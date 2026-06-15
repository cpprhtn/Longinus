# 🪪 identity/ — authentication & federation protocols

Identity protocols are where a small spec misunderstanding becomes full account takeover. This branch
covers **OAuth 2.0 / OIDC**, **JWT**, **SAML**, **MFA**, and **SSO** flows. Web-app login/session
mechanics (brute force, session fixation, password reset) live in
[../web/auth-and-session.md](../web/auth-and-session.md); this branch is the token/federation layer.
References: OAuth 2.0 Security BCP (RFC 9700), JWT BCP (RFC 8725), OpenID Connect.

> ⛔ Gate first. Token theft demos must use your own test accounts; never capture real users' tokens.

> 🛡️ **Enforce-forward.** Anchor to **NIST 800-63 / FAPI**. Class-killer: **library token-verify (pin
> alg + audience + exp) + mandatory `state`/PKCE**. Gate it with a **JWT-verify config lint/test** →
> [../enforce-forward.md](../enforce-forward.md) · [../templates/validation-layer.md](../templates/validation-layer.md).

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — JWT decode without algorithm pin**
```bash
rg -n "jwt\.decode\(|jwt\.verify\(|jose\." .
```
- **SKIP if:** `algorithms=[...]` is explicitly specified with a single algorithm family
- **SKIP if:** path contains `/test/`
- **FINDING if not skipped:** Type: JWT Algorithm Confusion | Severity: High | Fix: Pin algorithms to a single family (e.g., `algorithms=["RS256"]`)

**STEP 2 — JWT signature verification disabled**
```bash
rg -n "verify=False|verify_signature.*False|options.*verify.*false" .
```
- **SKIP if:** this is in logging/debugging code that never trusts the claims for auth decisions
- **FINDING if not skipped:** Type: JWT Signature Bypass | Severity: Critical | Fix: Always verify JWT signatures; never disable verification in production code

**STEP 3 — OAuth missing state parameter**
```bash
rg -n "authorize_url|authorization_endpoint|/authorize\?|oauth.*callback|redirect_uri" .
```
- **SKIP if:** a `state` parameter is generated and validated in the same flow
- **FINDING if not skipped:** Type: OAuth CSRF (missing state) | Severity: High | Fix: Generate a cryptographic random state parameter and validate it on callback

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## JWT (JSON Web Tokens) — test every claim and the signature

JWTs are tamper-evident *only if verified correctly*. Common, high-impact flaws:

- **`alg: none`** — server accepts an unsigned token. Strip the signature, set `"alg":"none"`, change
  `sub`/`role` → forge any identity. **Fix:** pin the expected algorithm; reject `none`.
- **Algorithm confusion (RS256 → HS256)** — server verifies with the *public* key but treats it as an
  HMAC secret; sign your forged token with the public key as the HMAC key. **Fix:** bind verification
  to one expected alg; never let the token's header pick the verification method.
- **Weak HMAC secret** — `HS256` with a guessable/short secret → crack offline (`hashcat -m 16500`),
  then forge. **Fix:** long random secret (≥256-bit), rotate, store as a secret.
- **Signature not actually verified** — app decodes but never validates (decode-only libraries, or
  catching verify errors). **Fix:** verify, fail closed.
- **Missing claim validation** — `exp`/`nbf`/`iat` not checked (replay forever), `aud`/`iss` not
  validated (token from another app accepted), `kid` injection (path traversal / SQLi via `kid` to
  control the key). **Fix:** validate `exp/nbf/iss/aud`; treat `kid` as untrusted (allowlist keys).
- **Sensitive data / over-trust in claims** — secrets in the (base64, *not* encrypted) payload; the
  server trusting a client-set `role`/`isAdmin` claim. **Fix:** don't put secrets in JWTs; derive
  authorization from server state, not client-modifiable claims.
- **No revocation** — stolen/long-lived tokens can't be killed. **Fix:** short TTL + refresh,
  server-side revocation list / token-binding for sensitive apps.

```
Test flow: decode header/payload → try alg:none → try RS256→HS256 confusion →
           tamper role/sub & resend → check exp/aud/iss enforcement → crack weak HS secret
```
(Deeper crypto attacks on tokens: [../cryptography/README.md](../cryptography/README.md).)

## OAuth 2.0 / OIDC — the flow is the attack surface

Map the exact flow (authorization-code/PKCE, implicit-legacy, client-credentials) and probe each
parameter:

- **`redirect_uri` validation** — the #1 OAuth bug. Test partial matches, open-redirect chaining,
  `redirect_uri=https://attacker`, path/subdomain/`@`/`#` tricks, missing exact-match. A loose
  `redirect_uri` → **auth code / token theft → ATO.** **Fix:** exact-match allowlist of full URIs.
- **`state` parameter** — missing/unvalidated `state` → **CSRF on the OAuth flow** (login CSRF /
  account linking takeover). **Fix:** unguessable `state`, bound to the session, verified on return.
- **PKCE** — missing/`plain` PKCE on public clients → code interception. **Fix:** enforce
  `S256` PKCE for public clients (OAuth 2.1 default).
- **Implicit flow / token in URL** — tokens leak via `Referer`/history/logs. **Fix:** use
  authorization-code + PKCE; retire implicit.
- **Scope & consent** — scope upgrade, missing consent re-prompt, mixing up `access_token` vs
  `id_token`, accepting an `access_token` where an `id_token` is required (or vice-versa).
- **Token/code reuse** — auth codes must be single-use & short-lived; refresh tokens rotated.
- **Account linking / pre-account-takeover** — sign-in-with-X linking by unverified email; "login with
  Google" matched to an existing local account by email without verification → ATO.
- **IdP confusion / `iss` mixups**, **JWKS/`jku`/`x5u` SSRF** (server fetches attacker-controlled key
  URL → forge tokens). **Fix:** pin/allowlist JWKS endpoints.

## SAML

- **Signature wrapping (XSW)** — restructure the XML so the app reads attacker-controlled assertions
  while the signature still validates a different element → forge identity. **Fix:** validate the
  signature covers the asserted element; use a hardened library; canonicalization care.
- **Unsigned/partially-signed assertions** accepted; **comment injection** in `NameID`
  (`admin<!---->@x` parsed as `admin`); **XXE** in the SAML XML parser (→ [../web/injection.md](../web/injection.md)).
- **Recipient/audience/`NotOnOrAfter`** not validated → replay/misuse. **Fix:** validate all
  conditions, recipient, audience, and freshness.

## MFA & step-up

- **MFA bypass:** is MFA enforced on *every* path (API, legacy endpoint, "remember device")? Can you
  skip the second step by going straight to the post-MFA endpoint? Is the OTP rate-limited (else
  brute-force the 6 digits)? Is the OTP reusable / not bound to the session? Backup-code abuse?
- **Enrollment flaws:** can an attacker enroll *their* MFA on a victim account during a reset?
- **Fix:** enforce MFA server-side on all auth paths, rate-limit + lock OTP attempts, single-use
  time-bound codes bound to the session, re-auth before MFA changes.

## DO NOT report as an identity vulnerability if…

- The JWT `decode` without verification is in **logging/debugging code** that never
  trusts the claims for authorization decisions
- The `redirect_uri` uses a **strict exact-match allowlist** you missed (check config,
  not just the comparison function)
- The "missing state" flow is **client-credentials** (machine-to-machine, no browser) —
  state is a browser-CSRF defense, not applicable to backend flows
- The "MFA bypass" requires **already having the password** and targets a user's own
  non-sensitive settings — assess actual impact before reporting
- The JWT has a short TTL and the app **uses refresh-token rotation** — "no revocation"
  with a 5-minute access token and rotated refresh is acceptable

---

## Vulnerable ↔ fixed code examples

### JWT alg:none bypass

```python
# ❌ VULNERABLE — accepts whatever algorithm the token claims (including none)
import jwt
def verify_token(token):
    payload = jwt.decode(token, PUBLIC_KEY, algorithms=["RS256", "HS256", "none"])
    return payload   # attacker strips signature, sets alg:"none" → forged identity

# ✅ FIXED — pin to the one expected algorithm
import jwt
def verify_token(token):
    payload = jwt.decode(token, PUBLIC_KEY, algorithms=["RS256"])  # only RS256 accepted
    return payload
```

### JWT RS256 → HS256 confusion

```javascript
// ❌ VULNERABLE — library picks algorithm from token header; accepts both
const decoded = jwt.verify(token, publicKey);
// attacker: signs token with HMAC using the *public key* as secret → verified!

// ✅ FIXED — explicitly pin the algorithm
const decoded = jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

### OAuth redirect_uri bypass

```python
# ❌ VULNERABLE — partial match allows attacker subdomains/paths
def validate_redirect(uri):
    return uri.startswith('https://app.example.com')
    # bypassed by: https://app.example.com.evil.com/steal
    # bypassed by: https://app.example.com@evil.com
    # bypassed by: https://app.example.com/../attacker-path

# ✅ FIXED — exact match against allowlist
ALLOWED_REDIRECTS = {
    'https://app.example.com/callback',
    'https://app.example.com/oauth/done'
}
def validate_redirect(uri):
    return uri in ALLOWED_REDIRECTS
```

### Missing state parameter (OAuth CSRF)

```javascript
// ❌ VULNERABLE — no state parameter → attacker can force-login victim to attacker's account
app.get('/auth/callback', async (req, res) => {
  const { code } = req.query;          // no state validation
  const token = await exchangeCode(code);
  loginUser(token);                     // attacker's code → victim logged into attacker's account
});

// ✅ FIXED — generate, store, and verify state
app.get('/auth/start', (req, res) => {
  const state = crypto.randomUUID();
  req.session.oauthState = state;
  res.redirect(`https://provider/authorize?...&state=${state}`);
});
app.get('/auth/callback', async (req, res) => {
  if (req.query.state !== req.session.oauthState) return res.sendStatus(403);
  delete req.session.oauthState;
  const token = await exchangeCode(req.query.code);
  loginUser(token);
});
```

---

## Static signs

```bash
rg -n "jwt\.(decode|verify)|algorithms?\s*[:=]|verify\s*[:=]\s*false|alg" -i .
rg -n "redirect_uri|response_type|grant_type|state\b|pkce|code_challenge" -i .
```
Red flags: `jwt.decode(token, verify=False)`, `algorithms` accepting multiple incl. `none`/both RS&HS,
`redirect_uri` compared with `startsWith`/`includes`, no `state` check, MFA checked only client-side.

## CTF angle

JWT `alg:none` / RS256→HS256 / weak-secret cracking to forge an `admin` token is one of the most common
web-CTF patterns; OAuth challenges hinge on `redirect_uri`/`state`; SAML challenges on signature
wrapping. Decode the token, attack the verification.

## Real-world cases

- **JWT library `alg` flaws (Auth0 / Tim McLean, 2015)** — "Critical vulnerabilities in JSON Web Token
  libraries": many libs let the *token's* header pick the verification algorithm → `alg:none` accepted,
  and RS256-verifying code tricked into HMAC-verifying with the public key. Still a top web-CTF and
  bug-bounty pattern. [Writeup](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/).
- **OAuth `redirect_uri` / `state` account-takeover** reports recur constantly on
  [HackerOne Hacktivity](https://hackerone.com/hacktivity) — loose redirect matching → auth-code/token
  theft → ATO.
- **Disclosed reports:** [TOP OAuth](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPOAUTH.md)
  · [account takeover](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPACCOUNTTAKEOVER.md)
  · [OpenID](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPOPENID.md)
  · [MFA bypass](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPMFA.md).

## References

[RFC 9700](https://datatracker.ietf.org/doc/rfc9700/) (OAuth 2.0 Security BCP) ·
[RFC 8725](https://www.rfc-editor.org/rfc/rfc8725) (JWT BCP) ·
[OIDC Core](https://openid.net/specs/openid-connect-core-1_0.html) ·
OWASP [JWT](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html) /
[SAML Security](https://cheatsheetseries.owasp.org/cheatsheets/SAML_Security_Cheat_Sheet.html) Cheat Sheets ·
PortSwigger [JWT](https://portswigger.net/web-security/jwt) / [OAuth](https://portswigger.net/web-security/oauth) Academy ·
CWE-[347](https://cwe.mitre.org/data/definitions/347.html)/[287](https://cwe.mitre.org/data/definitions/287.html)/[290](https://cwe.mitre.org/data/definitions/290.html).
Full bibliography: [research/identity.md](../../research/identity.md). Back to [tree](../00-map.md) ·
web-layer auth → [../web/auth-and-session.md](../web/auth-and-session.md).
