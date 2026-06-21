# GraphQL and API resource limits

GraphQL, search, export, and bulk endpoints can turn one request into huge server work. This covers
OWASP API4, API6, API9, and API10 where inventory, cost, and abuse controls matter.

## Mechanical scan

```bash
rg -n -i "graphql|apollo|GraphQLSchema|introspection|depth|complexity|DataLoader|limit|page_size|per_page|batch|alias|rateLimit|throttle|timeout" .
rg -n -i "/v1|/v2|/internal|/beta|swagger|openapi|api-docs|postman|\.proto|reflection" .
```

- **SKIP:** depth/complexity limits, pagination caps, body-size limits, timeouts, and rate limits are
  enforced for the relevant caller.
- **FINDING:** Unrestricted Resource Consumption | Severity: Medium-High | Fix: cost/depth/size/rate
  limits with per-identity quotas.
- **FINDING:** Improper API Inventory | Severity: Medium by default | Fix: inventory, retire old
  versions, isolate non-prod APIs.

## Confirm

1. Inventory the contract: OpenAPI, Postman, GraphQL introspection, gRPC reflection, client bundles.
2. For GraphQL, try deep nesting, batching/aliasing, and field-level authorization checks.
3. For REST/search/export, test large `limit`, `page_size`, broad filters, and expensive sorts.
4. Use tiny, bounded proofs. Show the control is absent; do not run a destructive load test.

## Fix

- Disable or intentionally expose introspection with auth-aware access.
- Enforce query depth, complexity, alias/batch caps, payload size, timeouts, and pagination max.
- Rate-limit by identity, tenant, IP, and expensive operation class.
- Maintain a versioned API inventory; retire shadow/zombie endpoints.
- Validate and sanitize third-party API responses before using them in sinks.

## Do not report if

- The endpoint is public, cheap, cached, and has no meaningful abuse impact.
- Limits exist at a gateway/service mesh and apply to the tested route.
- You only saw introspection; treat it as a finding only when it exposes sensitive schema or enables a
  concrete attack path.

Back to [api/](README.md).
