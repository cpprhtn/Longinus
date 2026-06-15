# Authentication & session management (A07:2025)

Can an attacker **get in** (bypass/guess/steal credentials), **stay in** (weak/forever sessions), or
**become someone else** (session fixation, takeover)? This leaf covers the web mechanics; protocol-level
identity (OAuth/OIDC/JWT/SAML/MFA) lives in [../identity/README.md](../identity/README.md). CWE-287,
CWE-384, CWE-613, CWE-307, CWE-620.

## Test surface

Login, signup, logout, "remember me", password reset/change, email/phone change, MFA enrollment &
verification, session cookies/tokens, account recovery, and any "act as" / impersonation feature.

## 1. Credential & login flaws

- **User enumeration:** different responses/timing for valid vs invalid usernames at login, signup,
  reset, or MFA. Confirm: compare messages, status, and response time. Fix: uniform responses and
  timing; generic "if an account exists, we sent an email."
- **Weak password policy / no breach check:** allows `password123`. Fix: length-first policy, block
  known-breached passwords (k-anonymity HIBP), no silly composition rules.
- **Brute force / credential stuffing:** no rate limit or lockout on login/MFA/reset. Confirm with a
  small, in-scope, throttled test (don't actually spray real accounts). Fix: rate limiting, lockout/
  backoff, CAPTCHA on anomaly, MFA, IP/device intelligence, breach-password blocking.
- **Default/seeded creds**, debug "login as" endpoints, and auth bypass via parameter (`?admin=true`,
  `role` in body — see [access-control.md](access-control.md)).

## 2. Session management

- **Token strength & generation:** session IDs must be long, random, unpredictable. Predictable/
  sequential tokens → hijack. Confirm by sampling tokens; fix: framework CSPRNG session store.
- **Session fixation:** does the session ID **rotate on login**? If a pre-auth token stays valid
  post-auth, an attacker who plants it can ride the victim's session. Confirm: set a known session,
  log in, see if the same ID is now authenticated. Fix: regenerate the session ID on every privilege
  change (login, step-up, role change).
- **Logout & lifetime:** does logout truly invalidate server-side? Do tokens expire? Idle + absolute
  timeouts? Are sessions revoked on password change? Confirm by replaying a token after logout/reset.
  Fix: server-side invalidation, sane timeouts, revoke-all-on-reset.
- **Cookie flags:** session cookies must be `HttpOnly`, `Secure`, `SameSite` (Lax/Strict), correct
  scope/path, no overly-broad `Domain`. (Details in [misconfiguration.md](misconfiguration.md) and
  [csrf.md](csrf.md).)
- **Concurrent sessions / device management:** can a user see/revoke active sessions? Are tokens bound
  to anything (device, IP-range optionally)?

## 3. Password reset & account recovery (high-yield)

This flow is a frequent takeover vector — test it hard:
- **Token predictability/leakage:** is the reset token random and single-use? Does it leak in the
  `Referer`, URL history, or to third-party scripts on the reset page?
- **Host header poisoning:** does the reset link's domain come from the `Host`/`X-Forwarded-Host`
  header? If so, an attacker can make the victim's email contain a link to *their* domain and capture
  the token. Confirm: change `Host` on the reset request, see if the email link follows. Fix: build
  reset URLs from a server-side configured base URL, never the request header.
- **No expiry / reuse / not invalidated** after use or after a new request; token tied to the wrong
  account; **IDOR on the reset** (`userId` in the body). 
- **Email/phone change** without re-auth or confirmation → silent takeover. Require current password /
  step-up MFA and confirm to the *old* address.

## 4. "Stay logged in" & tokens

- "Remember me" tokens: long-lived, must be revocable, single-use rotating, and not equivalent to the
  password. 
- API keys/PATs: scope, expiry, rotation (see [../identity/README.md](../identity/README.md) and
  [../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md)).

## Static signs (code audit)

```bash
rg -n "session\.regenerate|req\.session\.id|set_cookie|SESSION|jwt\.sign|bcrypt|argon2|pbkdf2|md5|sha1" -i .
```
Red flags: passwords hashed with MD5/SHA1/unsalted or fast hashes; no session regeneration on login;
reset URL built from `req.headers.host`; tokens with no expiry; secrets/JWT keys hard-coded.

## How to fix (summary)

- Strong, salted, slow password hashing (**argon2id** / **bcrypt** / **scrypt**); never reversible.
- Rate-limit + lockout/backoff + breach-password blocking; offer/enforce **MFA**.
- Rotate session ID on login; server-side invalidation; `HttpOnly`/`Secure`/`SameSite` cookies;
  idle+absolute timeouts; revoke on password reset.
- Reset tokens: CSPRNG, single-use, short TTL, server-built URL; re-auth for sensitive changes.
- Generic, uniform responses to avoid enumeration.

## CTF angle

Auth challenges: user enumeration → targeted brute, predictable reset tokens (timestamp/sequential),
JWT tampering ([../cryptography/README.md](../cryptography/README.md)), session-fixation, and
`Host`-header reset poisoning. Map the full reset flow before attacking it.

## References

[OWASP A07:2025](https://owasp.org/Top10/2025/) · WSTG-ATHN/SESS · CWE-[287](https://cwe.mitre.org/data/definitions/287.html)/[384](https://cwe.mitre.org/data/definitions/384.html)/[613](https://cwe.mitre.org/data/definitions/613.html)/[307](https://cwe.mitre.org/data/definitions/307.html) ·
OWASP [Authentication](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html) / [Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html) / [Forgot Password](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html) / [Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) Cheat Sheets ·
[PortSwigger: Authentication](https://portswigger.net/web-security/authentication). Full bibliography: [research/web.md](../../research/web.md). Back to [web/](README.md) ·
protocols → [../identity/README.md](../identity/README.md).
