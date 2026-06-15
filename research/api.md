# 🔌 api — references & tooling

Bibliography for the [`references/api/`](../references/api/README.md) branch (OWASP API Security Top
10:2023). ↑ back to the [RESEARCH hub](../RESEARCH.md).

## Frameworks & standards

- **OWASP API Security Top 10:2023** — API1 BOLA · API2 Broken Authentication · API3 Broken Object
  Property Level Authorization · API4 Unrestricted Resource Consumption · API5 BFLA · API6
  Unrestricted Access to Sensitive Business Flows · API7 SSRF · API8 Security Misconfiguration ·
  API9 Improper Inventory Management · API10 Unsafe Consumption of APIs. →
  https://owasp.org/API-Security/editions/2023/en/0x00-header/
- **OWASP GraphQL & REST Security Cheat Sheets.** →
  https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html ·
  https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- **WSTG-APIT** (API testing) and **CWE** classification — see [web.md](web.md) for WSTG/CWE base URLs.

## Tooling

> Dual-use: run only in scope (see [process.md](process.md) → authorization).

- API/GraphQL: [Postman](https://www.postman.com/) · [Hoppscotch](https://hoppscotch.io/) ·
  [graphw00f](https://github.com/dolevf/graphw00f) ·
  [clairvoyance](https://github.com/nikitastupin/clairvoyance) ·
  [graphql-cop](https://github.com/dolevf/graphql-cop) ·
  [kiterunner](https://github.com/assetnote/kiterunner) · [jwt_tool](https://github.com/ticarpi/jwt_tool)
- Proxy/all-rounder (shared with web): [Burp Suite](https://portswigger.net/burp) ·
  [OWASP ZAP](https://www.zaproxy.org/) · [mitmproxy](https://mitmproxy.org/) — full list in [web.md](web.md).
- Contract discovery (swagger/openapi/JS routes) → [recon.md](recon.md).

## Related references

[web.md](web.md) (sinks, proxies) · [identity.md](identity.md) (token/JWT/OAuth) ·
[recon.md](recon.md) (endpoint inventory) · [process.md](process.md) (severity, disclosure) ·
[meta-resources.md](meta-resources.md).
