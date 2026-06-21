# Identity protocols

Identity protocol mistakes turn small validation gaps into account takeover. This README is the
router for JWT, OAuth/OIDC, SAML, MFA, and SSO. Web login/session mechanics live in
[../web/auth-and-session.md](../web/auth-and-session.md).

> Gate first. Token theft demos must use your own test accounts; never capture real users' tokens.

## Mechanical scan

> Quick-mode consolidated identity scan. Route every hit to the exact leaf.

**STEP 1 - JWT verification** -> [jwt.md](jwt.md)
```bash
rg -n -i "jwt\.(decode|verify)|jose\.|algorithms?\s*[:=]|verify_signature|verify\s*[:=]\s*false|JWT_SECRET|HS256|RS256|kid|jwks|jku|x5u" .
```
- **SKIP:** decode-only use is logging/debugging and claims never drive authz
- **FINDING:** signature disabled **Critical**; alg confusion/weak secret/missing claims **High**

**STEP 2 - OAuth/OIDC flow** -> [oauth-oidc.md](oauth-oidc.md)
```bash
rg -n -i "redirect_uri|authorize_url|authorization_endpoint|oauth.*callback|state\b|pkce|code_challenge|response_type|id_token|access_token|issuer|jwks|client_secret" .
```
- **SKIP:** strict redirect allowlist, state validation, PKCE, and issuer/audience checks exist
- **FINDING:** missing state / loose redirect / token mix-up | **High-Critical**

**STEP 3 - SAML / MFA** -> [saml.md](saml.md), [mfa.md](mfa.md)
```bash
rg -n -i "saml|SAMLResponse|xmlsec|xml-crypto|passport-saml|Assertion|Audience|Recipient|NotOnOrAfter|mfa|totp|otp|webauthn|passkey|backup.?code|remember.?device|step.?up|2fa" .
```
- **SKIP:** hardened library validates the consumed assertion, all conditions, and server-side MFA state
- **FINDING:** SAML signature/condition bypass **High-Critical** | MFA bypass **High**

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## Leaf map

| Signal | Open |
|---|---|
| JWT decode/verify, `alg`, weak `JWT_SECRET`, `kid`, JWKS | [jwt.md](jwt.md) |
| OAuth/OIDC redirect, state, PKCE, issuer/audience, account linking | [oauth-oidc.md](oauth-oidc.md) |
| SAMLResponse, XML signature, assertion/audience/recipient/time conditions | [saml.md](saml.md) |
| OTP, TOTP, WebAuthn/passkeys, backup codes, remember-device, step-up | [mfa.md](mfa.md) |
| Login, password reset, session cookie, brute force | [../web/auth-and-session.md](../web/auth-and-session.md) |

## Enforce-forward

Anchor to NIST 800-63, OAuth/OIDC Security BCP, JWT BCP, and FAPI. The class-killer is library-based
token verification with pinned algorithms, audience/issuer/expiry checks, mandatory `state`/PKCE, and
server-side MFA state. Gate it with a JWT/identity config test in CI.

Back to [tree](../00-map.md).

References: [research/identity.md](../../research/identity.md).
