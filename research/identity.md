# 🪪 identity — references & tooling

Bibliography for the [`references/identity/`](../references/identity/README.md) branch (OAuth/OIDC,
JWT, SAML, MFA, SSO). ↑ back to the [RESEARCH hub](../RESEARCH.md).

## Frameworks & standards

- **OAuth 2.0 Security BCP (RFC 9700)**, **OAuth 2.0 (RFC 6749)** + **Bearer Token (RFC 6750)**,
  **OAuth 2.1** draft, **OpenID Connect Core**, **JWT BCP (RFC 8725)**, **JWT (RFC 7519)**. →
  https://datatracker.ietf.org/doc/rfc9700/ · https://www.rfc-editor.org/rfc/rfc8725 ·
  https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1 · https://openid.net/specs/openid-connect-core-1_0.html
- **OWASP JWT / Authentication / Session Management / OAuth2 / SAML Security cheat sheets.** →
  https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html ·
  https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html ·
  https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html ·
  https://cheatsheetseries.owasp.org/cheatsheets/SAML_Security_Cheat_Sheet.html
- **PortSwigger JWT & OAuth Academy.** → https://portswigger.net/web-security/jwt ·
  https://portswigger.net/web-security/oauth

## Tooling

- [jwt_tool](https://github.com/ticarpi/jwt_tool) (JWT attacks: `alg:none`, RS256→HS256 confusion,
  weak-secret cracking) · weak-HMAC cracking with [hashcat](https://hashcat.net/hashcat/) (mode 16500)
  / [John the Ripper](https://www.openwall.com/john/) — see [ctf.md](ctf.md).
- Proxy + the web toolset apply (login/session flows are web too) → [web.md](web.md).

## Related references

[web.md](web.md) (web-layer auth/session) · [api.md](api.md) (token-protected APIs) ·
[ctf.md](ctf.md) (JWT/crypto attacks, `alg` confusion, HMAC cracking) · [process.md](process.md) ·
[meta-resources.md](meta-resources.md).
