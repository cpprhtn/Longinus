# 🔗 Mass data-exfiltration chains

The highest-frequency real-world breach: not RCE, but **reading everyone's data** by composing an
authorization gap with automation. Parent: [../chaining-and-impact.md](../chaining-and-impact.md).

## Chain 1 — IDOR/BOLA + predictable IDs + no rate limit → full dump

1. **IDOR/BOLA**: an object endpoint (`/api/users/{id}`, `/orders/{id}`) returns *another* user's object
   with no ownership check → [../web/access-control.md](../web/access-control.md) · [../api/README.md](../api/README.md).
2. IDs are **sequential/guessable** (or leaked elsewhere — UUIDs in a public list).
3. **No rate limit** → script the full range → **entire dataset exfiltrated**. (The classic single
   highest-impact API bug.)

## Chain 2 — excessive data exposure (read the JSON, not the UI)

1. The API returns **more fields than the UI renders** (`passwordHash`, `ssn`, internal flags, other
   users' PII) → [../api/README.md](../api/README.md).
2. Read raw responses → harvest secrets/PII the front-end hid → often feeds an **ATO** chain
   ([account-takeover.md](account-takeover.md)).

## Chain 3 — GraphQL batching/aliasing → amplified extraction

1. GraphQL **introspection** on → map hidden fields/types.
2. **Batched/aliased** queries (1000 aliases in one request) bypass per-request rate limits + deeply
   nested queries amplify → bulk-extract or DoS → [../api/README.md](../api/README.md).

## Chain 4 — blind injection / SSRF → out-of-band exfil

1. A **blind** SQLi/NoSQLi or **SSRF** with no response body → [../web/injection.md](../web/injection.md) · [../web/ssrf.md](../web/ssrf.md).
2. Exfiltrate via **timing** or an **OOB channel** (DNS/HTTP callback encoding the stolen data) → slow
   but total read.

## Real-world cases

[TOP IDOR](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPIDOR.md)
· [TOP AUTHORIZATION](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPAUTHORIZATION.md)
· [TOP GRAPHQL](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPGRAPHQL.md)
— BOLA-driven mass extraction dominates API breaches.

## Prove each link

⛔ APIs make bulk extraction trivial — a **2–3 record** cross-account proof is the PoC; **never**
exfiltrate the real dataset ([../authorization-and-scope.md](../authorization-and-scope.md)). Stay
non-destructive and within scope.

## 🛡️ Defensive seal

- **Per-request, per-object ownership check scoped to the caller** (kills Chain 1 even with guessable IDs).
- **Response DTOs that allowlist fields per role** — never `return user` (Chain 2).
- **GraphQL: disable introspection in prod, depth/complexity + batch limits, field-level authz** (Chain 3).
- **Rate-limit + anomaly detection + egress allowlist** blunts the automation/OOB step (Chains 1, 4).

→ [../enforce-forward.md](../enforce-forward.md) (API row) · [../templates/validation-layer.md](../templates/validation-layer.md).

Back to [chain catalog](../chaining-and-impact.md) · [tree](../00-map.md).
