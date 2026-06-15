# Cross-Site Request Forgery (CSRF)

The attacker's site makes the victim's browser send a **state-changing request** to your app, riding
the victim's logged-in cookies. The app can't tell the forged request from a real one because the
browser auto-attaches cookies. Impact = whatever the action does (change email/password, transfer
funds, change settings → account takeover). CWE-352.

> CSRF only matters when auth is **ambient** (cookies, HTTP basic, client certs). Pure
> `Authorization: Bearer` header APIs are largely immune *because the browser won't auto-add the
> header* — but watch hybrid apps that accept a cookie fallback.

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — State-changing handlers without CSRF protection**
```bash
rg -n "app\.(post|put|delete|patch)\(|router\.(post|put|delete|patch)\(" .
```
- **SKIP if:** the application uses Bearer tokens (not cookies) for auth — CSRF is not applicable
- **SKIP if:** CSRF middleware is applied globally
- **FINDING if not skipped:** Type: CSRF | Severity: Medium | Fix: Add CSRF token validation to all state-changing endpoints

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## Where it hides

Any state-changing endpoint without anti-CSRF protection: password/email change, settings, money
movement, role grants, content CRUD, "delete account", admin actions. Especially:
- endpoints that accept **`GET`** for state changes (always wrong),
- forms/JSON endpoints with **no CSRF token** or a token that isn't validated,
- **`SameSite=None`** (or absent, on older browsers) session cookies.

## How to find it

1. Capture a state-changing request. Check for an anti-CSRF token (form field / custom header).
2. **Remove or alter the token** and replay:
   - drop it entirely; use an empty value; use *another user's* token; reuse an old token.
   - if it still succeeds → no/weak CSRF protection.
3. **Method/content-type tricks:** change `POST`→`GET`; switch JSON to
   `application/x-www-form-urlencoded`/`text/plain` (simple requests that skip CORS preflight) and see
   if the endpoint still accepts it.
4. **SameSite check:** inspect the session cookie's `SameSite`. `Strict`/`Lax` blocks most cross-site
   sends (Lax still allows top-level GET navigations — so a state-changing GET is exploitable even with
   Lax).
5. Build a PoC HTML page that auto-submits the forged request; load it while authenticated in another
   tab; confirm the action executed.

## How to confirm (PoC)

```html
<!-- minimal CSRF PoC: auto-submits to the target as the logged-in victim -->
<form action="https://target.example/account/email" method="POST">
  <input name="email" value="attacker@evil.test">
</form>
<script>document.forms[0].submit()</script>
```
Show the victim's account state changed (email updated) purely from visiting the attacker page. For a
GET-based CSRF, a single `<img src="https://target/...action...">` suffices.

## How to fix

- **Use the framework's CSRF protection** — almost every framework ships it; the bug is usually that
  it was disabled or the endpoint opted out. Prefer:
  - **Synchronizer token** (per-session/per-request token validated server-side), or
  - **Double-submit cookie** (token in cookie + header, compared), or signed/HMAC tokens.
- **`SameSite` cookies:** set session cookies to `SameSite=Lax` (or `Strict` for sensitive apps) as a
  strong baseline — but don't rely on it *alone* (Lax allows top-level GET; older clients vary).
- **Never use `GET` for state changes.** Require `POST`/`PUT`/`DELETE`.
- **Validate Origin/Referer** for state-changing requests as defense-in-depth.
- **Require re-authentication / step-up** for the most sensitive actions (password, email, payment).
- For JSON APIs, require a **custom header** (e.g. `X-Requested-With` or a CSRF header) that cross-site
  simple requests can't set, and don't accept `text/plain`/form content-types for JSON endpoints.

```js
// ✅ example: csurf-style middleware + SameSite session cookie
app.use(csrf());                         // validates token on state-changing methods
res.cookie('sid', id, { httpOnly:true, secure:true, sameSite:'lax' });
```

## Related / chains

- **Login CSRF:** force the victim into the *attacker's* account (then they enter data the attacker
  reads). Fix: CSRF-protect the login form too.
- **CSRF + self-XSS → stored XSS**; **CSRF + weak CORS** (see [misconfiguration.md](misconfiguration.md))
  can read responses, turning a blind CSRF into data theft.
- **Clickjacking** is the UI-redress cousin — defend with `X-Frame-Options: DENY` /
  `frame-ancestors 'none'` (see [misconfiguration.md](misconfiguration.md)).

## Real-world cases

Disclosed HackerOne reports with PoCs:
[CSRF](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPCSRF.md).

## References

WSTG-SESS-05 · [CWE-352](https://cwe.mitre.org/data/definitions/352.html) ·
[OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html) ·
[PortSwigger: CSRF](https://portswigger.net/web-security/csrf). Full bibliography: [research/web.md](../../research/web.md). Back to [web/](README.md).
