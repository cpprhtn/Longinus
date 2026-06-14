# 🌐 web — references & tooling

Bibliography for the [`references/web/`](../references/web/README.md) branch (OWASP Top 10:2025).
↑ back to the [RESEARCH hub](../RESEARCH.md).

## Frameworks & standards

- **OWASP Top 10:2025** — finalized Jan 2026, derived from 175k+ CVEs. Categories:
  A01 Broken Access Control · A02 Security Misconfiguration · A03 Software Supply Chain Failures ·
  A04 Cryptographic Failures · A05 Injection · A06 Insecure Design · A07 Authentication Failures ·
  A08 Software or Data Integrity Failures · A09 Security Logging & Alerting Failures ·
  A10 Mishandling of Exceptional Conditions. New in 2025: A03 (supply chain) and A10 (exceptional
  conditions); SSRF was absorbed into A01. → https://owasp.org/Top10/2025/ ·
  prior edition for mapping: https://owasp.org/Top10/ (2021)
- **OWASP Web Security Testing Guide (WSTG)** — the canonical "how to test" checklist (WSTG-INFO,
  -CONF, -IDNT, -ATHN, -ATHZ, -SESS, -INPV, -ERRH, -CRYP, -BUSL, -CLNT, -APIT). →
  https://owasp.org/www-project-web-security-testing-guide/
- **OWASP ASVS (Application Security Verification Standard)** — verification requirements by level. →
  https://owasp.org/www-project-application-security-verification-standard/
- **OWASP Proactive Controls** — the defensive counterpart (what to build, not just what to break). →
  https://owasp.org/www-project-proactive-controls/
- **OWASP Cheat Sheet Series** — per-topic defensive references (cited by name throughout `web/`). →
  https://cheatsheetseries.owasp.org/ (index: https://cheatsheetseries.owasp.org/Glossary.html).
  Leaf-cited sheets resolve to `…/cheatsheets/<Name>_Cheat_Sheet.html`, e.g.
  [SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html),
  [XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html),
  [CSP](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html),
  [SSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html),
  [CSRF Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html),
  [Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html),
  [IDOR Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html),
  [Deserialization](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html),
  [File Upload](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html).
- **OWASP Secure Headers Project** — recommended HTTP response headers + the `securityheaders.com`
  scanner (cited in `web/misconfiguration.md`). → https://owasp.org/www-project-secure-headers/ ·
  https://securityheaders.com/
- **PortSwigger Web Security Academy** — free labs + reference for every web bug class. →
  https://portswigger.net/web-security (per-topic: `/access-control`, `/sql-injection`,
  `/cross-site-scripting`, `/ssrf`, `/csrf`, `/authentication`, `/file-path-traversal`, `/file-upload`,
  `/deserialization`, `/jwt`, `/oauth`, `/web-cache-poisoning`, `/request-smuggling`).
- **CWE (Common Weakness Enumeration)** — weakness taxonomy used for classification. →
  https://cwe.mitre.org/ ; CWE Top 25 → https://cwe.mitre.org/top25/ ; individual:
  `https://cwe.mitre.org/data/definitions/<N>.html`.

## Tooling

> Dual-use: run only in scope (see [process.md](process.md) → authorization). Tools surface
> *candidates*, not findings — confirm manually.

- Proxy/all-rounder: [Burp Suite](https://portswigger.net/burp) · [OWASP ZAP](https://www.zaproxy.org/) ·
  [mitmproxy](https://mitmproxy.org/) · [Caido](https://caido.io/)
- Injection/specific: [sqlmap](https://sqlmap.org/) ·
  [commix](https://github.com/commixproject/commix) · [tplmap](https://github.com/epinna/tplmap) ·
  [dalfox](https://github.com/hahwul/dalfox) · [interactsh](https://github.com/projectdiscovery/interactsh) ·
  [NoSQLMap](https://github.com/codingo/NoSQLMap)
- Deserialization gadgets: [ysoserial](https://github.com/frohoff/ysoserial) (Java) ·
  [ysoserial.net](https://github.com/pwntester/ysoserial.net) (.NET) · [PHPGGC](https://github.com/ambionics/phpggc) (PHP)
- Client-side / hardening: [DOMPurify](https://github.com/cure53/DOMPurify) (HTML sanitizer, defensive) ·
  [securityheaders.com](https://securityheaders.com/) (header scanner)
- Desync & cache (advanced): [HTTP Request Smuggler / Param Miner](https://github.com/portswigger/param-miner)
  (Burp) · [Turbo Intruder](https://github.com/portswigger/turbo-intruder) (single-packet race) ·
  [smuggler](https://github.com/defparam/smuggler) · [h2csmuggler](https://github.com/BishopFox/h2csmuggler) ·
  [Web-Cache-Vulnerability-Scanner](https://github.com/Hackmanit/Web-Cache-Vulnerability-Scanner) ·
  [Singularity](https://github.com/nccgroup/singularity) (DNS rebinding).
- Prototype pollution: Burp **DOM Invader** ·
  [client-side gadget DB](https://github.com/BlackFan/client-side-prototype-pollution) ·
  [PPScan](https://github.com/msrkp/PPScan).
- API/GraphQL tooling lives in [api.md](api.md); recon/content-discovery in [recon.md](recon.md).

## Advanced / modern research (top-tier bounty & CTF)

The classes that still pay top bounties at mature programs and define hard CTF web — track the edge and
master these:

- **[PortSwigger Top 10 Web Hacking Techniques](https://portswigger.net/research/top-10-web-hacking-techniques)**
  — the annual community-voted list of the year's most important new techniques. Read it every year.
- **Request smuggling / desync (James Kettle):**
  [HTTP Desync Attacks](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn) ·
  [HTTP/2: The Sequel is Always Worse](https://portswigger.net/research/http2) ·
  [Browser-Powered Desync Attacks](https://portswigger.net/research/browser-powered-desync-attacks)
  → leaf [../references/web/request-smuggling-and-desync.md](../references/web/request-smuggling-and-desync.md).
- **Web cache:** [Practical Web Cache Poisoning](https://portswigger.net/research/practical-web-cache-poisoning) ·
  [Web Cache Entanglement](https://portswigger.net/research/web-cache-entanglement).
- **Race conditions:** [Smashing the State Machine](https://portswigger.net/research/smashing-the-state-machine)
  (the single-packet attack).
- **Prototype pollution:** [Server-side prototype pollution](https://portswigger.net/research/server-side-prototype-pollution)
  · client-side gadgets [BlackFan](https://github.com/BlackFan/client-side-prototype-pollution)
  → leaf [../references/web/prototype-pollution.md](../references/web/prototype-pollution.md).
- **Full feed:** [PortSwigger Research](https://portswigger.net/research). Disclosed reports & CTF
  writeups: [meta-resources.md](meta-resources.md).

## Further reading (curated lists)

- [qazbnm456/awesome-web-security](https://github.com/qazbnm456/awesome-web-security) — web-security
  materials & resources. Bug-bounty tooling/methodology → [recon.md](recon.md).

## Related references

[api.md](api.md) · [identity.md](identity.md) (JWT/OAuth) · [ctf.md](ctf.md) (crypto/TLS, web-CTF) ·
[secrets-supply-chain.md](secrets-supply-chain.md) (A03/A08) · [process.md](process.md) (methodology,
severity, disclosure) · meta & living-doc policy: [meta-resources.md](meta-resources.md).
