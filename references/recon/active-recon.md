# Active recon (probing the live target)

Now you send traffic to the target. **This requires authorization** and adherence to the program's
rate limits and allowed hours (see [../authorization-and-scope.md](../authorization-and-scope.md)).
Use test markers so your traffic is identifiable. Default to non-destructive — discovery, not exploit.

## 1. Resolve & validate the asset list

Turn the passive list into live hosts:
```bash
# resolve subdomains, then check which are alive
dnsx -l subs.txt -resp -silent | tee resolved.txt
httpx -l subs.txt -sc -title -tech-detect -cdn -ip -silent | tee live.txt
```
`httpx` gives you status code, page title, detected tech, CDN/WAF presence, and IP — the backbone of
your live inventory. Flag anything unusual (dev/staging/internal-sounding hostnames, non-standard
ports, missing CDN = possible origin).

## 2. Active subdomain expansion

- **Brute force:** resolve a wordlist of candidate names (`shuffledns`/`puredns` + a good list) against
  the target's resolvers.
- **Permutation:** `dev-`, `staging-`, `api-`, `-old`, numeric suffixes (`gotator`/`dnsgen`/`altdns`).
- **Subdomain takeover check:** dangling CNAMEs pointing to unclaimed cloud resources (S3, GitHub
  Pages, Heroku, Azure) — `nuclei -t takeovers`, `subjack`, `nuclei` DNS templates. A confirmed
  takeover is a real, often High finding.

## 3. Port & service scanning

```bash
# fast wide sweep, then targeted deep scan
naabu -host target -top-ports 1000 -silent          # or masscan for speed at scale
nmap -sV -sC -p<found-ports> target                  # service/version + default scripts
```
Identify exposed services beyond HTTP (SSH, DBs, Redis, Elasticsearch, RDP, SMB, mail). Unauthenticated
databases/caches (Redis, Mongo, Elastic, Memcached with no auth) are classic critical finds. Respect
scope — only scan authorized ranges, and throttle to avoid availability impact.

## 4. Content & directory discovery

Find unlinked endpoints, files, and admin panels:
```bash
ffuf -u https://target/FUZZ -w wordlist.txt -mc 200,204,301,302,401,403 -fs <baseline-size>
feroxbuster -u https://target -w wordlist.txt -x php,bak,old,zip,env,json
```
- Use **content-length / response-word filtering** to cut noise from soft-404s.
- Hunt for: `/.git/`, `/.env`, `/admin`, `/api`, `/swagger`, `/actuator`, `/.well-known/`, backups
  (`.bak/.old/.zip/.sql`), source maps (`.js.map`).
- A reachable **`.git/`** directory → dump the whole repo (`git-dumper`) → source + secrets. High value.

## 5. Crawl & JavaScript analysis

Modern apps hide most of their surface in JS:
```bash
katana -u https://target -jc -kf all -silent          # crawl incl. JS endpoints
# extract endpoints/secrets from JS bundles
```
- Pull endpoints, parameters, and API routes from JS (`linkfinder`/`xnLinkFinder`-style).
- Look for hard-coded keys, internal hostnames, feature flags, and `.map` files exposing source.
- Parse `swagger.json`/`openapi.json`/GraphQL introspection for the full API contract (→ [../api/README.md](../api/README.md)).

## 6. Parameter discovery & fuzzing

- **Find hidden params:** `arjun`, `x8`, or fuzz `FUZZ=value` against a known endpoint and watch for
  behavioral/response-size changes.
- **Mine historical params** (from `gau`/`waybackurls`) and replay them on live endpoints.
- These parameters are the raw material for the [web/](../web/README.md) injection/SSRF/IDOR tests.

## 7. Vulnerability-surface tagging

- **Version → CVE:** map fingerprinted tech versions to known CVEs (don't blindly trust banners —
  confirm exploitability before reporting).
- **Templated checks:** `nuclei` (CVEs, exposures, misconfigs, takeovers, default creds) — high signal
  when you curate templates; review every hit, don't paste raw scanner output into a report.
- **Default credentials** on admin panels/devices (only test if in scope; one careful attempt, not a
  brute-force spray).

## Discipline

- **Throttle.** Honor rate limits; never let scanning degrade availability (that edges into DoS).
- **Mark your traffic.** Identifiable `User-Agent`/header per the ROE.
- **In scope only.** Discovery may reveal out-of-scope assets — note, don't probe.
- **Confirm, then hand off.** Output is a prioritized, fingerprinted endpoint list for the testing
  playbooks — not a finding by itself.

## References

[OWASP WSTG (Information Gathering / Config & Deploy testing)](https://owasp.org/www-project-web-security-testing-guide/) ·
[PTES — Intelligence Gathering](http://www.pentest-standard.org/) ·
[NIST SP 800-115](https://csrc.nist.gov/pubs/sp/800/115/final) (technical testing). Tool URLs & full
bibliography: [research/recon.md](../../research/recon.md). Disclosed reports:
[subdomain takeover on HackerOne](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPSUBDOMAINTAKEOVER.md).
⛔ Active probing requires authorization —
[../authorization-and-scope.md](../authorization-and-scope.md).

Tools: [../tooling/README.md](../tooling/README.md). Next: [../web/README.md](../web/README.md) ·
[../api/README.md](../api/README.md). Back to [recon/](README.md).
