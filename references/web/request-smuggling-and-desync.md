# Request smuggling, desync & web cache attacks (advanced)

The elite end of web exploitation: disagreements about **where one HTTP request ends** let you prepend
bytes to *another user's* request, poison shared caches, or bypass front-end controls. These are the
bugs that win PortSwigger's "Top 10 Web Hacking Techniques" and pay top-tier bounties at large orgs
with layered proxies/CDNs. High-impact, easy to *break things* — test only in scope, throttle, and
prefer self-targeted proofs. CWE-444 (HTTP request/response smuggling), CWE-444/441, CWE-525 (cache).

> ⚠️ Smuggling can poison requests of *real users*. On live targets use a benign collaborator payload
> and target **your own** session/connection; never queue a payload that harms other users. Gate first.

## Sub-classes

- **Classic request smuggling (HTTP/1.1)** — front-end and back-end disagree on request length:
  - **CL.TE** — front-end uses `Content-Length`, back-end uses `Transfer-Encoding: chunked`.
  - **TE.CL** — the reverse.
  - **TE.TE** — both support TE but one is tricked by an obfuscated header (`Transfer-Encoding : chunked`,
    `Transfer-Encoding: xchunked`, dual TE, tab/space tricks).
  - **CL.0 / H2.0** — back-end ignores the body length entirely (treats `CL` as 0).
- **HTTP/2 desync** — downgrade/rewrite issues when a HTTP/2 front-end talks HTTP/1.1 to the back-end:
  **H2.CL**, **H2.TE**, CRLF injection in HTTP/2 header names/values, request **tunnelling**.
- **Client-side / browser-powered desync** — the desync lives in a single connection the *victim's
  browser* makes (no shared back-end needed); enables stored/reflected desync and "pause"-based attacks.
- **h2c smuggling** — upgrade to cleartext HTTP/2 to bypass a reverse proxy's routing/authz.
- **Web cache poisoning** — get a harmful response stored under a key other users share (unkeyed
  headers/params, fat GET, cache-key normalization quirks) → mass XSS/redirect/DoS.
- **Web cache deception** — trick the cache into storing a victim's *authenticated* response under a
  cacheable-looking path (`/account.php/nonexistent.css`) → read their data.

## Where it hides

Anywhere there's a **chain of HTTP processors**: CDN → WAF → load balancer → app server, reverse
proxies, API gateways, service meshes. The more hops (and the more HTTP/2↔1.1 translation), the more
desync surface. Caches: CDNs (Cloudflare/Akamai/Fastly), Varnish, nginx cache, framework caches.

## How to find it

**Smuggling — the detection methodology (timing-based, safe):**
1. Send a request crafted so that *if* the back-end mis-parses, it waits for more bytes → a **time delay**
   appears. This confirms a desync *without* affecting other users.
2. Use **Burp Suite → HTTP Request Smuggler** extension (Albinowax) — it automates CL.TE/TE.CL/TE.TE/
   CL.0/H2 probes and the timing technique. Or **[smuggler](https://github.com/defparam/smuggler)**.
3. For HTTP/2: disable Burp's auto-update of `Content-Length`, allow malformed headers, test H2.CL/H2.TE,
   CRLF in header names (`foo: bar\r\nx: y`), and request tunnelling.
4. **h2c:** [h2csmuggler](https://github.com/BishopFox/h2csmuggler) to attempt an upgrade past the proxy.

**Cache poisoning:** find **unkeyed inputs** (headers like `X-Forwarded-Host`, `X-Forwarded-Scheme`,
`X-Host`, params, cookies) that influence the response but aren't in the cache key — use **Param Miner**
(Burp). Confirm the poisoned response is served to a clean request. Scanner:
[Web-Cache-Vulnerability-Scanner](https://github.com/Hackmanit/Web-Cache-Vulnerability-Scanner).

**Cache deception:** request an authenticated page with an appended cacheable extension/path segment
(`/profile/foo.css`, `/profile;foo.js`, `/profile%2f%2e%2e`) and see if a second, unauthenticated fetch
of the same URL returns *your* private data.

## How to confirm (PoC)

- **Smuggling:** prove you can prepend to the *next* request on the connection — e.g. smuggle a request
  whose effect (a 4xx/redirect, or your benign header reflected) lands on a follow-up request you send
  on the same connection. Capturing another arbitrary user is the impact; **demonstrate the primitive,
  don't harvest victims.** Use a Collaborator/OOB marker.
- **Cache poisoning:** poison with a benign marker (e.g. a harmless reflected header → a unique comment),
  then show a clean request receives it from cache. State the blast radius (every cached user).
- **Cache deception:** show the cached URL serves your authenticated data to an anonymous client.

## How to fix

- **Speak one HTTP version end-to-end** (HTTP/2 all the way, no downgrade), or normalize/reject ambiguous
  messages: drop requests with both `CL` and `TE`, reject obfuscated `TE`, reject CR/LF in HTTP/2 fields.
- Use a front-end that **rejects** (not forwards) malformed length headers; disable connection reuse to
  the back-end where feasible; prefer back-ends that close on ambiguity.
- **Cache:** key on every input that changes the response; never cache authenticated responses; decide
  cacheability by content-type/route allowlist, not URL suffix; strip caching for paths that 404→200
  rewrite. Set `Cache-Control: no-store` on personalized responses.

## CTF angle

Less common in classic CTF, but appears in advanced web/"realistic" tracks: chain smuggling to bypass a
front-end auth check and reach an internal `/admin`/`/flag`, or poison a cache to land XSS on the admin
bot. The methodology mirrors the labs in PortSwigger Academy.

## Real-world cases

- **James Kettle (PortSwigger) research lineage** — the canonical primary sources:
  [HTTP Desync Attacks (2019)](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn) ·
  [HTTP/2: The Sequel is Always Worse (2021)](https://portswigger.net/research/http2) ·
  [Browser-Powered Desync Attacks (2022)](https://portswigger.net/research/browser-powered-desync-attacks) ·
  [Practical Web Cache Poisoning (2018)](https://portswigger.net/research/practical-web-cache-poisoning) ·
  [Web Cache Entanglement (2020)](https://portswigger.net/research/web-cache-entanglement).
- **Disclosed reports:** [TOP request smuggling](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPREQUESTSMUGGLING.md)
  · [TOP web cache](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPWEBCACHE.md) on HackerOne.

## References

CWE-[444](https://cwe.mitre.org/data/definitions/444.html) · WSTG-INPV (HTTP splitting/smuggling) ·
[PortSwigger: Request smuggling](https://portswigger.net/web-security/request-smuggling) /
[Web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning) /
[Web cache deception](https://portswigger.net/web-security/web-cache-deception) ·
tools: [HTTP Request Smuggler / Param Miner](https://github.com/portswigger/param-miner) ·
[smuggler](https://github.com/defparam/smuggler) · [h2csmuggler](https://github.com/BishopFox/h2csmuggler).
Advanced research index: [research/web.md](../../research/web.md). Back to [web/](README.md).
