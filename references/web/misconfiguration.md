# Security misconfiguration (A02:2025) + headers, CORS, errors, logging

Moved up to **#2 in 2025** — and the most common class in vibe-coded/cloud-deployed apps because the
defaults are insecure and nobody hardened them. Individually often Low; in chains, decisive. Also
covers **A09:2025 (Logging & Alerting Failures)** and the response/error side of **A10:2025**. CWE-16,
CWE-1004, CWE-942, CWE-209, CWE-548, CWE-778.

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — Debug mode enabled**
```bash
rg -n "DEBUG\s*=\s*True|debug:\s*true|NODE_ENV.*development|app\.debug|EnableDetailedErrors" .
```
- **SKIP if:** this is in a development/test config only, not production
- **FINDING if not skipped:** Type: Debug Mode Enabled | Severity: Medium | Fix: Disable debug mode in production configuration

**STEP 2 — Permissive CORS**
```bash
rg -n "Access-Control-Allow-Origin.*\*|cors\(\)|allow_origins.*\*|origin.*true" .
```
- **SKIP if:** the endpoint serves only public, non-authenticated data
- **FINDING if not skipped:** Type: Permissive CORS | Severity: Medium | Fix: Restrict allowed origins to specific trusted domains

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## Checklist (sweep these fast)

### Exposed files & endpoints
- `/.git/`, `/.svn/`, `/.env`, `/.DS_Store`, `config.*`, `*.bak/.old/.zip/.sql`, source maps (`.js.map`)
- `/swagger`, `/openapi.json`, GraphQL introspection, `/actuator` (Spring), `/debug`, `/metrics`,
  `/.well-known/`, admin panels, phpinfo, server-status
- directory listing enabled; backup/editor temp files; `robots.txt` revealing sensitive paths
- **Confirm:** fetch them; a reachable `.git` → `git-dumper` → full source+secrets (High).

### Defaults & hardening
- default/guessable credentials on admin panels, DBs, dashboards (Grafana, Jenkins, Kibana, Redis,
  Mongo with no auth)
- debug mode on in prod (Django `DEBUG=True`, Flask debugger/Werkzeug console → RCE, stack traces)
- sample/test apps, unused features/ports/services enabled
- verbose version banners (fingerprint → CVE mapping)

### Security headers (test every response)
| Header | Want | Why |
|---|---|---|
| `Content-Security-Policy` | strict, nonce/hash, no `unsafe-inline` | XSS mitigation |
| `Strict-Transport-Security` | `max-age` long, `includeSubDomains` | force HTTPS |
| `X-Content-Type-Options` | `nosniff` | MIME-sniffing XSS |
| `X-Frame-Options` / CSP `frame-ancestors` | `DENY`/`'none'` | clickjacking |
| `Referrer-Policy` | `no-referrer`/`strict-origin...` | token/URL leakage |
| `Set-Cookie` | `Secure; HttpOnly; SameSite` | session theft / CSRF |
| `Cache-Control` | `no-store` on sensitive responses | cached secrets |
| `Permissions-Policy` | least-privilege features | feature abuse |
Check with `securityheaders.com`, `nuclei` headers templates, or manual inspection. Missing headers are
hardening findings; weak/absent cookie flags or a permissive CSP can be the *enabler* in an XSS/CSRF
chain.

### CORS (often a real bug, not just hardening)
- **Find:** send `Origin: https://evil.test` and inspect the response:
  - reflecting arbitrary `Access-Control-Allow-Origin` **with** `Access-Control-Allow-Credentials: true`
    → **any site can read authenticated responses** = credential/data theft (High).
  - `ACAO: *` on authenticated endpoints, `null` origin trusted, weak regex (`example.com.evil.test`,
    `evilexample.com`), trusting subdomains that allow user content.
- **Confirm (non-destructive):** from a benign attacker origin, show a cross-origin `fetch(...,
  {credentials:'include'})` reading a victim-scoped response (use your own account).
- **Fix:** strict allowlist of exact origins; never reflect arbitrary `Origin`; never combine `*` (or
  reflected origin) with credentials; validate against a fixed list, not a regex.

### TLS / crypto config (→ A04:2025)
- weak protocols/ciphers (SSLv3/TLS1.0/1.1), expired/mismatched/self-signed certs, no HSTS, mixed
  content, missing forward secrecy. Test with `testssl.sh`/`sslscan`. Deeper crypto:
  [../cryptography/README.md](../cryptography/README.md).

### Error handling & info disclosure (A10:2025 response side)
- stack traces, framework/version, SQL errors, internal IPs/paths, exception messages leaked to users
- different errors enabling user/resource enumeration (see [auth-and-session.md](auth-and-session.md))
- **Fix:** generic error pages, log details server-side only, disable debug in prod, consistent
  responses.

### Logging, alerting & monitoring (A09:2025)
- security events **not** logged (logins, failures, access-control denials, high-value actions)
- logs without integrity/forwarding; no alerting on anomalies; secrets/PII written to logs (also a
  leak — see [../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md))
- **Fix:** log security-relevant events with enough context (who/what/when, no secrets), centralize,
  alert on thresholds, protect log integrity, retain appropriately.

## Static signs
```bash
rg -n "DEBUG\s*=\s*True|app\.debug\s*=\s*true|NODE_ENV.*develop|displayErrorDetails" -i .
rg -n "Access-Control-Allow-Origin|cors\(|allowedOrigins|ACAO" -i .
rg -n "console\.log\(.*password|logger\.(info|debug)\(.*token" -i .   # secrets in logs
```

## How to fix (systemic)

- **Harden by default, automate it.** Security baseline as code (IaC + CIS Benchmarks — see
  [../cloud-and-infra/README.md](../cloud-and-infra/README.md)); identical hardened config across
  dev/stage/prod.
- **Minimize surface:** remove unused features, ports, services, default accounts, sample apps; block
  access to VCS/dotfiles/source maps at the web server/CDN.
- **Set security headers globally** (helmet/secure-headers middleware or at the edge/CDN).
- **Strict CORS allowlist; never reflect Origin + credentials.**
- **No debug in prod; generic errors; structured security logging with alerting.**
- **Continuously scan config** (nuclei, trivy/checkov for IaC) in CI so drift is caught.

## CTF angle

Exposed `.git`/`.env`/source maps reveal the flag or creds; debug consoles (Werkzeug PIN, Spring
Actuator `/env`+`/heapdump`) give RCE/secret dumps; permissive CORS or a leaked `swagger` reveals the
path to the flag. First thing in a web challenge: check for exposed config/VCS/debug endpoints.

## Real-world cases

Disclosed HackerOne reports with PoCs:
[information disclosure](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPINFODISCLOSURE.md) ·
[web cache poisoning/deception](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPWEBCACHE.md).

## References

[OWASP A02/A09/A10:2025](https://owasp.org/Top10/2025/) · WSTG-CONF/ERRH · CWE-[16](https://cwe.mitre.org/data/definitions/16.html)/[942](https://cwe.mitre.org/data/definitions/942.html)/[209](https://cwe.mitre.org/data/definitions/209.html)/[548](https://cwe.mitre.org/data/definitions/548.html)/[778](https://cwe.mitre.org/data/definitions/778.html) ·
[OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/) · [CORS](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html) / [Error Handling](https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html) / [Logging](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html) Cheat Sheets ·
[securityheaders.com](https://securityheaders.com/). Full bibliography: [research/web.md](../../research/web.md). Back to [web/](README.md).
