# API object and function authorization (BOLA / BFLA)

APIs fail most often when authentication proves who the caller is, but authorization does not prove
which object or function the caller may use. Covers OWASP API1 and API5.

## Mechanical scan

```bash
# Object lookups driven by request IDs
rg -n "findById\(|findByPk\(|\.get\(.*params\.|Model\.objects\.get\(pk=|where.*id.*req\.params|params\.(id|userId|orderId)" .

# Admin / privileged routes and verbs
rg -n -i "admin|manage|delete|disable|approve|role|permission|@require|require_role|isAdmin|is_staff" .
```

- **SKIP:** query is scoped by caller/tenant (`user_id`, `owner_id`, `org_id`, `tenant_id`) or a
  verified middleware/guard enforces that exact object/function permission.
- **FINDING:** BOLA/BFLA | Severity: High by default; Critical if unauthenticated or cross-tenant at
  scale | Fix: deny by default; scope every object query and enforce role/permission per function.

## Confirm

1. Build an endpoint inventory from routes, OpenAPI/Postman, GraphQL schema, or client bundles.
2. Use two accounts or tenants. Fetch object A as owner, then replay the same request as user B.
3. For functions, call admin/management endpoints as a normal user; try sibling verbs and old API
   versions (`/v1` vs `/v2`).
4. A 2-3 record proof is enough. Do not bulk-extract real data.

## Fix

- Put ownership in the query: `WHERE id = ? AND tenant_id = session.tenant_id`.
- Enforce function permission on every route/resolver/service method, not only in the UI or gateway.
- Use 404 for inaccessible objects when existence should not leak.
- Add contract tests: user A cannot read/write user B's object; normal user cannot call admin action.

## Do not report if

- The ownership/role check exists in a global guard, route group, policy layer, or query scope you
  initially missed.
- The endpoint only returns public data and has no state-changing or sensitive effect.
- You cannot demonstrate unauthorized access or action; park as Needs-validation.

Back to [api/](README.md).
