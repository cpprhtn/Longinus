# API security (OWASP API Security Top 10:2023)

APIs are most apps' real attack surface. They fail mostly at authorization, property-level trust,
resource controls, and inventory. This README is the router; open one leaf, not the whole branch.
For protocol identity (JWT/OAuth/SAML/MFA) see [../identity/](../identity/README.md); for web sinks
see [../web/](../web/README.md).

> Gate first. APIs make bulk extraction easy: a 2-3 record proof is enough.

## Mechanical scan

> Quick-mode consolidated API scan. Run these greps, apply skip conditions, then route hits to the
> linked leaf for confirmation.

**STEP 1 - BOLA/BFLA: object or function authorization** -> [bola-bfla.md](bola-bfla.md)
```bash
rg -n "findById\(|findByPk\(|\.get\(.*params\.|Model\.objects\.get\(pk=|where.*id.*req\.params|params\.(id|userId|orderId)" .
rg -n -i "admin|manage|delete|disable|approve|role|permission|@require|require_role|isAdmin|is_staff" .
```
- **SKIP:** ownership/tenant scope or role/permission guard is verified on that path
- **FINDING:** BOLA/BFLA | **High** by default; **Critical** for unauth/cross-tenant scale

**STEP 2 - Mass assignment / excessive data exposure** -> [mass-assignment-data-exposure.md](mass-assignment-data-exposure.md)
```bash
rg -n "Object\.assign\(.*req\.body|User\(\*\*request\.data\)|\.update\(req\.body\)|findByIdAndUpdate\(.*req\.body|model_validate\(.*request|from_dict\(.*request" .
rg -n "\.to_dict\(\)|\.toJSON\(\)|res\.json\(user\)|jsonify\(.*\.query\.|serialize.*all|__dict__|model_to_dict" .
```
- **SKIP:** request/response DTO allowlists fields per role
- **FINDING:** Mass Assignment **High** | Excessive Data Exposure **Medium-High**

**STEP 3 - GraphQL/resource limits/inventory** -> [graphql-resource-limits.md](graphql-resource-limits.md)
```bash
rg -n -i "graphql|apollo|introspection|depth|complexity|limit|page_size|batch|alias|rateLimit|throttle|timeout|/v1|/v2|/internal|/beta|swagger|openapi|api-docs|\.proto|reflection" .
```
- **SKIP:** cost/depth/size/rate limits and inventory controls are present for the tested route
- **FINDING:** Resource Consumption / Improper Inventory | **Medium-High**

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## Leaf map

| Signal | Open |
|---|---|
| Object ID, tenant ID, user ID, admin function, privileged verb | [bola-bfla.md](bola-bfla.md) |
| `req.body` bound to model, `role/isAdmin/ownerId/price` accepted, full model returned | [mass-assignment-data-exposure.md](mass-assignment-data-exposure.md) |
| GraphQL, batching/aliasing, large limits, old API versions, shadow endpoints | [graphql-resource-limits.md](graphql-resource-limits.md) |
| JWT, OAuth, SAML, MFA, SSO | [../identity/](../identity/README.md) |
| SQLi/SSRF/XSS/deserialization/file upload in API handlers | [../web/](../web/README.md) |

## OWASP API Top 10 mapping

| ID | Risk | Longinus leaf |
|---|---|---|
| API1 | Broken Object Level Authorization | [bola-bfla.md](bola-bfla.md) |
| API2 | Broken Authentication | [../web/auth-and-session.md](../web/auth-and-session.md), [../identity/](../identity/README.md) |
| API3 | Broken Object Property Level Authorization | [mass-assignment-data-exposure.md](mass-assignment-data-exposure.md) |
| API4 | Unrestricted Resource Consumption | [graphql-resource-limits.md](graphql-resource-limits.md) |
| API5 | Broken Function Level Authorization | [bola-bfla.md](bola-bfla.md) |
| API6 | Sensitive business flows | [graphql-resource-limits.md](graphql-resource-limits.md), [../web/business-logic.md](../web/business-logic.md) |
| API7 | SSRF | [../web/ssrf.md](../web/ssrf.md) |
| API8 | Security Misconfiguration | [../web/misconfiguration.md](../web/misconfiguration.md) |
| API9 | Improper Inventory Management | [graphql-resource-limits.md](graphql-resource-limits.md) |
| API10 | Unsafe Consumption of APIs | [graphql-resource-limits.md](graphql-resource-limits.md) |

Back to [tree](../00-map.md).

References: [research/api.md](../../research/api.md).
