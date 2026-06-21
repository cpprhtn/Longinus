# Basic vibe-code triage

Use this after the authorization gate and the secrets 60-second scan, before opening a deep domain
leaf. It catches common high-yield failures with a small, stack-agnostic pass. Treat every hit as a
lead: apply the skip rules, then route to the exact leaf for confirmation.

## Mechanical scan

```bash
# Default / seeded / hard-coded credentials
rg -n -i "(admin|root|test|demo).{0,24}(admin|1234|password|pass|changeme)|password.{0,12}['\"](admin|1234|password|test|demo|changeme)['\"]|create_superuser|is_superuser|seed.*admin|demo.*user" .

# Debug, admin, and exposed internal surfaces
rg -n -i "debug\s*[:=]\s*true|DEBUG\s*=\s*True|/admin|/dashboard|/swagger|/api-docs|/openapi|/debug|/actuator|/metrics|phpinfo" .

# Client-controlled privilege / business fields
rg -n -i "req\.(body|query|params).*(isAdmin|role|permission|userId|ownerId|tenantId|price|total|balance)|request\.(data|args|json).*(is_admin|role|user_id|owner_id|tenant_id|price|total|balance)" .

# Weak app secrets and permissive cross-origin config
rg -n -i "JWT_SECRET\s*[:=]\s*['\"](secret|changeme|dev|test)|SESSION_SECRET\s*[:=]\s*['\"](secret|changeme|dev|test)|cors\(\)|allow_origins.*\*|Access-Control-Allow-Origin.*\*" .

# Exposed files likely to contain source, config, DBs, or backups
git ls-files | rg -n -i "(\.env|\.sqlite|\.db|\.sql|backup|dump|\.bak|\.old|\.zip|id_rsa|\.pem|\.p12|credentials|\.npmrc|\.pypirc)"
```

## How to report / route

| Signal | Type | Default severity | Open next |
|---|---|---|---|
| Real default credential or seeded admin in production path | Default / seeded credentials | High | [web/auth-and-session.md](web/auth-and-session.md) |
| Debug/admin/internal endpoint exposed outside dev | Security misconfiguration | Medium-High | [web/misconfiguration.md](web/misconfiguration.md) |
| `role`, `isAdmin`, `ownerId`, `tenantId`, `price`, `total` trusted from client input | Authz / mass assignment / business logic | High | [api/mass-assignment-data-exposure.md](api/mass-assignment-data-exposure.md) |
| Weak JWT/session secret | Weak signing/session secret | High | [identity/jwt.md](identity/jwt.md) or [web/auth-and-session.md](web/auth-and-session.md) |
| Committed env, DB, backup, key, or credential file | Secret / exposed artifact | High-Critical | [secrets-and-supply-chain/secret-detection.md](secrets-and-supply-chain/secret-detection.md) |

## Skip / downgrade

- Skip obvious examples in docs, tests, fixtures, mocks, or generated sample apps unless they are
  shipped or enabled in production.
- Skip placeholder values such as `your-api-key-here`, `example-password`, or fake test tokens.
- Downgrade debug/admin surfaces if they are bound only to localhost or a documented dev-only profile.
- Do not file client-controlled fields from a grep alone; confirm the server actually trusts them and
  no DTO/allowlist/role check removes them.

Back to [tree](00-map.md).
