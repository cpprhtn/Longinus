# JWT verification and claim trust

JWTs are tamper-evident only when the application verifies the signature and validates the claims it
trusts. Covers RFC 8725 / JWT BCP issues.

## Mechanical scan

```bash
rg -n -i "jwt\.(decode|verify)|jose\.|algorithms?\s*[:=]|verify_signature|verify\s*[:=]\s*false|JWT_SECRET|HS256|RS256|kid|jwks|jku|x5u" .
```

- **SKIP:** decode-only use is logging/debugging and the claims never drive authz.
- **FINDING:** Signature verification disabled | Severity: Critical | Fix: verify signatures and fail
  closed.
- **FINDING:** Algorithm confusion / weak secret / missing claim validation | Severity: High | Fix:
  pin algorithm family, use strong secrets/keys, validate `exp`, `nbf`, `iss`, `aud`.

## Test flow

1. Decode header/payload; look for `alg`, `kid`, `iss`, `aud`, `exp`, role/admin claims.
2. Try `alg:none` only in an owned/lab target.
3. Check RS256/HS256 confusion when both families are accepted.
4. Tamper `sub`, `role`, or `isAdmin`; confirm the server rejects it.
5. For HS256, assess whether the secret is hard-coded or guessable; do not crack live third-party
   tokens outside scope.
6. Check `kid`/`jku`/`x5u` for path traversal, SSRF, or attacker-controlled JWKS.

## Fix

- Pin one expected algorithm family; never let the token header choose verification behavior.
- Validate `exp`, `nbf`, `iat`, `iss`, and `aud`; reject missing critical claims.
- Keep signing keys in a secret manager; rotate; use long random HS secrets or asymmetric keys.
- Derive authorization from server-side state where possible; do not trust client-set role claims.
- Use short access-token TTLs plus refresh-token rotation and revocation for sensitive apps.

Back to [identity/](README.md).
