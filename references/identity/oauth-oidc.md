# OAuth 2.0 / OIDC flow security

OAuth/OIDC bugs are flow bugs: a loose redirect, missing state, or token mix-up can become account
takeover. Covers OAuth 2.0 Security BCP and OIDC.

## Mechanical scan

```bash
rg -n -i "redirect_uri|authorize_url|authorization_endpoint|oauth.*callback|state\b|pkce|code_challenge|response_type|id_token|access_token|issuer|jwks|client_secret" .
```

- **SKIP:** machine-to-machine client-credentials flows do not need browser `state`.
- **FINDING:** Missing/unvalidated `state` | Severity: High | Fix: random state bound to session and
  verified on callback.
- **FINDING:** Loose `redirect_uri` validation | Severity: High-Critical when token/code theft is
  possible | Fix: exact-match allowlist.
- **FINDING:** Missing PKCE on public client / implicit flow | Severity: Medium-High | Fix:
  authorization-code + PKCE S256.

## Test flow

1. Map the exact grant type and callback path.
2. Test redirect validation with partial matches, subdomains, `@`, path traversal, fragments, and open
   redirect chaining.
3. Confirm `state` is generated, stored, bound to the browser session, and checked once.
4. Check PKCE (`S256`) for public clients and mobile/SPA flows.
5. Verify issuer/audience and token type: do not accept an access token where an ID token is required.
6. Test account linking by unverified email and IdP confusion.

## Fix

- Exact-match registered redirect URIs.
- Mandatory `state`; PKCE S256 for public clients.
- Retire implicit flow; use authorization-code + PKCE.
- Validate issuer, audience, nonce, token type, code single-use, and refresh rotation.
- Pin/allowlist JWKS endpoints; never fetch attacker-supplied key URLs.

Back to [identity/](README.md).
