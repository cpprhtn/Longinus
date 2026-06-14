# 🌐 web/ — web application security

The biggest, highest-yield branch — and where most vibe-coded apps bleed. This README maps the
**OWASP Top 10:2025** to the leaf playbooks and gives you a testing order. Test methodology follows
the **OWASP Web Security Testing Guide (WSTG)**; defensive fixes reference the **OWASP Cheat Sheets**.

> ⛔ Gate first ([../authorization-and-scope.md](../authorization-and-scope.md)). For your own code,
> static-audit freely; for live targets, stay in scope and non-destructive.

## OWASP Top 10:2025 → Longinus leaves

| 2025 | Category | Where to test it |
|---|---|---|
| **A01** | Broken Access Control *(SSRF folded in here in 2025)* | [access-control.md](access-control.md) · [ssrf.md](ssrf.md) |
| **A02** | Security Misconfiguration | [misconfiguration.md](misconfiguration.md) |
| **A03** | Software Supply Chain Failures | [../secrets-and-supply-chain/dependency-supply-chain.md](../secrets-and-supply-chain/dependency-supply-chain.md) |
| **A04** | Cryptographic Failures | [../cryptography/README.md](../cryptography/README.md) · [misconfiguration.md](misconfiguration.md) (TLS/cookies) |
| **A05** | Injection *(incl. XSS)* | [injection.md](injection.md) · [xss.md](xss.md) |
| **A06** | Insecure Design | [business-logic.md](business-logic.md) |
| **A07** | Authentication Failures | [auth-and-session.md](auth-and-session.md) · [../identity/README.md](../identity/README.md) |
| **A08** | Software or Data Integrity Failures | [deserialization.md](deserialization.md) · [../secrets-and-supply-chain/dependency-supply-chain.md](../secrets-and-supply-chain/dependency-supply-chain.md) |
| **A09** | Security Logging & Alerting Failures | [misconfiguration.md](misconfiguration.md) (logging/monitoring section) |
| **A10** | Mishandling of Exceptional Conditions | [business-logic.md](business-logic.md) · [misconfiguration.md](misconfiguration.md) (error handling) |

Plus cross-cutting leaves: [csrf.md](csrf.md), [file-upload-and-path.md](file-upload-and-path.md).
**Advanced / research-grade** (top-tier bounty & CTF): [request-smuggling-and-desync.md](request-smuggling-and-desync.md),
[prototype-pollution.md](prototype-pollution.md).

## Full leaf list

| Leaf | Covers |
|---|---|
| [access-control.md](access-control.md) | IDOR/BOLA, vertical/horizontal priv-esc, forced browsing, path traversal, mass assignment |
| [injection.md](injection.md) | SQL, NoSQL, OS command, LDAP, XPath, SSTI, XXE, header/CRLF, log injection |
| [xss.md](xss.md) | Reflected / stored / DOM XSS, CSP bypass, mutation XSS |
| [ssrf.md](ssrf.md) | SSRF: cloud metadata, internal pivot, blind/OOB, filter bypasses |
| [auth-and-session.md](auth-and-session.md) | Login flaws, credential attacks, session fixation/management, password reset |
| [csrf.md](csrf.md) | CSRF, SameSite gaps, state-changing GET, JSON/CORS interplay |
| [file-upload-and-path.md](file-upload-and-path.md) | Unrestricted upload, LFI/RFI, path traversal, zip-slip, image/SVG tricks |
| [deserialization.md](deserialization.md) | Insecure deserialization across Java/PHP/Python/.NET/Node/Ruby |
| [business-logic.md](business-logic.md) | Workflow abuse, race conditions, price/quantity tampering, IDOR-adjacent logic |
| [misconfiguration.md](misconfiguration.md) | Headers, CORS, cookies, debug/defaults, verbose errors, exposed files, logging |
| [request-smuggling-and-desync.md](request-smuggling-and-desync.md) | HTTP/1.1 smuggling (CL.TE/TE.CL/TE.TE/CL.0), HTTP/2 desync, browser-powered desync, h2c, web cache poisoning & deception |
| [prototype-pollution.md](prototype-pollution.md) | Client & server-side prototype pollution, source→gadget chains → XSS/RCE/SSRF/auth-bypass |

## Recommended testing order

Follow WSTG roughly top-to-bottom, but lead with what scales worst when broken:

1. **Map first.** Authenticated + unauthenticated crawl; enumerate roles, endpoints, parameters,
   state-changing actions, and where input lands.
2. **Access control** ([access-control.md](access-control.md)) — A01 is #1 for a reason; IDOR/BOLA and
   missing function-level checks are the most common *and* highest-impact bugs in CRUD apps. Test every
   object reference across roles.
3. **Authentication & session** ([auth-and-session.md](auth-and-session.md)) — can you get in, stay in,
   or become someone else?
4. **Injection & XSS** ([injection.md](injection.md), [xss.md](xss.md)) — every input → every sink.
5. **SSRF** ([ssrf.md](ssrf.md)) — any feature that fetches a URL.
6. **CSRF / file upload / deserialization** — where the features exist.
7. **Business logic** ([business-logic.md](business-logic.md)) — the bugs scanners can't find.
8. **Misconfiguration** ([misconfiguration.md](misconfiguration.md)) — headers, CORS, errors, exposed
   files; quick wins and chain-enablers.
9. **Advanced / research-grade** — when the target has CDNs/proxies or a JS-heavy/Node stack, escalate
   to [request-smuggling-and-desync.md](request-smuggling-and-desync.md) and
   [prototype-pollution.md](prototype-pollution.md). These are where mature programs still pay top
   bounties: the front-end/back-end gap and JS gadget chains that scanners and most hunters miss.
   Track the field via PortSwigger's "Top 10 Web Hacking Techniques" ([research/web.md](../../research/web.md)).

## Per-leaf shape

Every leaf is structured the same so you can move fast:
**What it is · Where it hides · How to find it (manual + tooling) · How to confirm (PoC) · How to fix ·
CTF angle · References.**

Tools: [../tooling/README.md](../tooling/README.md) · Methodology: [../methodology.md](../methodology.md).
WSTG: https://owasp.org/www-project-web-security-testing-guide/ · Cheat Sheets:
https://cheatsheetseries.owasp.org/
