# Server-Side Request Forgery (SSRF) — A01:2025

You make the **server** issue a request to a destination *you* choose. Because that request originates
from inside the target's network, it reaches things you can't: cloud metadata endpoints, internal
admin panels, databases, and the loopback interface — frequently chaining into **cloud credential
theft and full account takeover.** OWASP folded SSRF into A01 (Broken Access Control) in 2025. CWE-918.

## Where it hides

Any feature where the server fetches a URL/host you influence:
- webhooks, "import from URL", URL preview/unfurl, PDF/screenshot/thumbnail generators,
- avatar/image fetch-by-URL, file upload "from link", RSS/feed readers,
- integrations that take a callback/endpoint, SSO/OIDC metadata URLs, XML/SVG parsers (XXE→SSRF),
- partial-SSRF in headers (`Host`, `X-Forwarded-Host`), proxies, and PDF/HTML renderers.

Grep (static):
```bash
rg -n "requests\.(get|post)\(|axios\.|fetch\(|http\.get|urllib|HttpClient|file_get_contents|curl_exec" .
# flag any of the above whose URL/host comes from req/params/body
```

## How to find it

1. Identify the URL/host parameter. Point it at a **collaborator you control** (Burp Collaborator,
   `interactsh`, a DNS canary). A DNS/HTTP hit proves the server fetched your URL → SSRF.
2. Pivot to **internal targets** (only as deep as authorized):
   - loopback: `http://127.0.0.1:<port>`, `http://localhost`, `http://[::1]`
   - cloud metadata: AWS `http://169.254.169.254/latest/meta-data/` (and IMDSv2 token flow),
     GCP `http://metadata.google.internal/` (`Metadata-Flavor: Google`),
     Azure `http://169.254.169.254/metadata/instance?api-version=...` (`Metadata: true`)
   - internal services: `http://127.0.0.1:6379` (Redis), `:9200` (Elasticsearch), `:2375` (Docker)
3. **Blind SSRF** (no response body): rely on OOB callbacks and timing; escalate via gopher/redis/
   protocol smuggling only with explicit authorization.

> Default non-destructive proof: a benign OOB callback or reaching a harmless internal banner is enough
> to demonstrate SSRF. **Do not** actually steal cloud credentials on a target you don't own — report
> the reachability and stop. On your own infra/lab, demonstrate the full chain.

## Filter-bypass arsenal (for testing a *defense*)

Naïve blocklists fail to many encodings — test whether the app's filter is one of them:
- alternate IP encodings: `http://2130706433/` (decimal), `0x7f.0.0.1`, `0177.0.0.1` (octal),
  `127.1`, `[::ffff:127.0.0.1]`
- DNS rebinding (a hostname that resolves to a public IP first, then to `127.0.0.1`)
- redirect-based: a permitted host that 302s to an internal URL
- URL parser confusion: `http://expected.com@127.0.0.1/`, `http://127.0.0.1#expected.com`,
  `http://expected.com\@127.0.0.1`
- `0.0.0.0`, link-local, and cloud-metadata DNS names

These belong in a report as "here's why your allowlist is insufficient," with the fix below.

## How to confirm (PoC)

- OOB: unique callback received at your collaborator from the target's egress IP.
- In-band: response contains content only reachable internally (internal title, metadata structure,
  a service banner) — with a marker proving it came from your injected URL.

## How to fix

- **Allowlist destinations** (scheme + host + port) — *never* a blocklist. Resolve the hostname,
  validate the **resolved IP** against the allowlist, and **re-validate after redirects** (and pin to
  the validated IP to defeat DNS rebinding / TOCTOU).
- **Block internal ranges** at the app *and* network layer: loopback, RFC1918, link-local
  (`169.254.0.0/16`), `0.0.0.0`, IPv6 ULAs/`::1`/mapped-v4.
- **Disable unneeded URL schemes** (`file:`, `gopher:`, `dict:`, `ftp:`).
- **Lock down cloud metadata:** enforce **IMDSv2** (session-token, hop-limit) on AWS; restrict
  metadata access; least-privilege the instance role so a leaked credential is low-value.
- **Don't return the fetched response** to the user (prevents in-band exfil); strip/normalize the
  request; isolate the fetcher (egress proxy, separate network segment, no internal route).

```python
# ✅ resolve, validate the IP, then fetch the pinned IP
ip = socket.gethostbyname(host)
if is_private_or_metadata(ip): abort(400)
requests.get(url, allow_redirects=False, timeout=5)   # and re-check on any redirect
```

## Advanced / high-impact techniques

- **Cloud metadata across providers** (the crown-jewel SSRF target): AWS
  `http://169.254.169.254/latest/meta-data/iam/security-credentials/` — defeat **IMDSv2** by abusing a
  *full* SSRF that can send the token `PUT` (or find legacy IMDSv1 hosts); **GCP**
  `http://metadata.google.internal/computeMetadata/v1/` (needs header `Metadata-Flavor: Google` — reachable
  if you can inject headers); **Azure** `http://169.254.169.254/metadata/instance?api-version=2021-02-01`
  (needs `Metadata: true`); **k8s** `https://kubernetes.default` + kubelet `:10250`.
  [HackTricks: cloud SSRF](https://book.hacktricks.wiki/en/pentesting-web/ssrf-server-side-request-forgery/cloud-ssrf.html).
- **Protocol smuggling:** `gopher://` forges arbitrary TCP payloads (Redis/Memcached/FastCGI/SMTP/
  unauthenticated DBs → **RCE**); also `dict://`, `file://`, `ftp://`. Turns "fetch a URL" into "speak
  to any internal service."
- **DNS rebinding:** pass a host allowlist with a domain you control, then re-resolve it to
  `169.254.169.254`/an internal IP after the check (TOCTOU) — [Singularity of Origin](https://github.com/nccgroup/singularity).
- **Blind SSRF → impact:** with no response body, still hit internal *state-changing* endpoints, trigger
  deserialization, or exfiltrate via timing/OOB; chain CRLF injection in the fetched URL to smuggle a
  request to the internal service.

## CTF angle

SSRF challenges typically gate the flag behind an "internal-only" endpoint (`/admin`, `/flag`,
`127.0.0.1:internal`) reachable only via the server's fetch feature, or behind a cloud-metadata mock.
Bypass the host filter, hit the internal route, read the flag.

## Real-world cases

- **Capital One (2019)** — an SSRF in a WAF reached the EC2 metadata service (IMDSv1) at
  `169.254.169.254/latest/meta-data/iam/security-credentials/`, stole the instance role's temporary
  creds, and exfiltrated ~106M records from S3. The reason **IMDSv2** (token-bound) exists.
  [Technical writeup](https://blog.appsecco.com/an-ssrf-privileged-aws-keys-and-the-capital-one-breach-4c3c2cded3af).
- **Orange Tsai — "A New Era of SSRF" (Black Hat 2017)** — URL-parser inconsistencies + CR-LF to slip
  past host allowlists; the canonical filter-bypass research.
  [Slides](https://blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf)
  · case collection: [AllThingsSSRF](https://github.com/jdonsec/AllThingsSSRF).
- **Disclosed reports:** [TOP SSRF on HackerOne](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPSSRF.md)
  (PoCs, incl. cloud-metadata chains).

## References

[OWASP A01:2025](https://owasp.org/Top10/2025/) (SSRF) · API7:2023 · WSTG-INPV-19 ·
[CWE-918](https://cwe.mitre.org/data/definitions/918.html) ·
[OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html) ·
[PortSwigger: SSRF](https://portswigger.net/web-security/ssrf). Full bibliography: [research/web.md](../../research/web.md). Back to [web/](README.md).
