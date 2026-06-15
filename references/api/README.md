# 🔌 api/ — API security (OWASP API Security Top 10:2023)

APIs are most apps' real attack surface — the UI is just one client. They fail differently from
classic web apps: the bugs are overwhelmingly **authorization** (who can touch which object/function)
and **resource/inventory** problems, not injection. This branch maps the **OWASP API Security Top
10:2023**. For protocol identity (OAuth/JWT) see [../identity/README.md](../identity/README.md); for
web sinks see [../web/README.md](../web/README.md).

> ⛔ Gate first. APIs make bulk-extraction trivially easy — be extra careful to stay non-destructive
> and not exfiltrate real data; a 2–3 record proof suffices.

## Mechanical scan — API quick checks

Run these before the deeper analysis below.

1. Build endpoint inventory from route definitions
2. For each endpoint with an object ID: is ownership checked? → BOLA sweep
3. Are admin endpoints guarded by role/permission middleware? → BFLA sweep
4. Check response serialization: do endpoints return full model objects? → data exposure
5. Is there rate limiting on auth endpoints (login, OTP, reset)?

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — BOLA: Object lookup without ownership**
```bash
rg -n "findById\(|findByPk\(|\.get\(.*params\.|Model\.objects\.get\(pk=" .
```
- **SKIP if:** same handler includes ownership filter (`user_id`, `owner_id`, `org_id`, `tenant_id`)
- **SKIP if:** path contains `/test/`
- **FINDING if not skipped:** Type: BOLA (Broken Object Level Authorization) | Severity: High | Fix: Scope query by authenticated user (e.g., `AND user_id = ?`)

**STEP 2 — Mass assignment**
```bash
rg -n "Object\.assign\(.*req\.body|User\(\*\*request\.data\)|\.update\(req\.body\)|findByIdAndUpdate\(.*req\.body" .
```
- **SKIP if:** a DTO or allowlist filters fields before assignment
- **FINDING if not skipped:** Type: Mass Assignment | Severity: High | Fix: Use an allowlist of updatable fields before binding request body

**STEP 3 — Full object serialization (excessive data exposure)**
```bash
rg -n "\.to_dict\(\)|\.toJSON\(\)|res\.json\(user\)|serialize.*all|jsonify\(.*\.query\.|__dict__" .
```
- **SKIP if:** a response DTO or serializer explicitly excludes sensitive fields
- **FINDING if not skipped:** Type: Excessive Data Exposure | Severity: Medium | Fix: Use explicit response DTOs that include only allowed fields

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## Discover the API contract first

You can't test endpoints you don't know. Build the full inventory:
- `swagger.json` / `openapi.json` / `/api-docs`, Postman collections, GraphQL **introspection**
  (`__schema`), gRPC reflection, `.proto` files.
- Mine endpoints/params from the SPA's JS bundles and historical URLs (see [../recon/](../recon/active-recon.md)).
- Enumerate **versions** (`/v1`, `/v2`, `/internal`, `/beta`) and **undocumented** routes — old
  versions are often less protected (API9).

## OWASP API Security Top 10:2023

| ID | Risk | How to test | Fix |
|---|---|---|---|
| **API1** | **BOLA** (Broken Object Level Authorization) | swap object IDs across accounts/roles; ~40% of API attacks. See [../web/access-control.md](../web/access-control.md) | per-request ownership check scoped to the caller; unpredictable IDs as DiD |
| **API2** | Broken Authentication | weak/again-able tokens, JWT flaws, no rate limit on login/OTP, credential stuffing | strong auth, short-lived signed tokens, MFA, rate limit; see [../identity/](../identity/README.md) |
| **API3** | Broken Object **Property** Level Authorization | **mass assignment** (set `role`/`isAdmin`/`balance`) + **excessive data exposure** (API returns fields the UI hides) | allowlist readable & writable fields per role; response DTOs, never `return user` |
| **API4** | Unrestricted Resource Consumption | no rate/size/page limits; expensive queries; large `limit=`; GraphQL deep/batched queries → DoS & cost | rate limits, pagination caps, query cost/depth limits, payload size caps, timeouts |
| **API5** | **BFLA** (Broken Function Level Authorization) | call admin/privileged functions as a low-priv user; toggle method/verb; guess admin routes | enforce role on every function server-side; deny by default |
| **API6** | Unrestricted Access to Sensitive Business Flows | automate a flow meant to be human-paced (buy-all-stock, mass signup, scalping, spam) | bot/abuse protection, per-identity throttling, see [../web/business-logic.md](../web/business-logic.md) |
| **API7** | **SSRF** | URL params the server fetches → internal/metadata. See [../web/ssrf.md](../web/ssrf.md) | allowlist destinations, block internal ranges, IMDSv2 |
| **API8** | Security Misconfiguration | missing headers, permissive CORS, verbose errors, no TLS, debug. See [../web/misconfiguration.md](../web/misconfiguration.md) | harden by default, automate config checks |
| **API9** | Improper Inventory Management | shadow/zombie/old-version endpoints, non-prod APIs exposed, undocumented hosts | API inventory/governance, retire old versions, separate non-prod |
| **API10** | Unsafe Consumption of 3rd-party APIs | trusting upstream data, following its redirects, injecting its responses into sinks | validate/sanitize upstream data, don't blindly trust integrations |

## High-yield API testing moves

1. **BOLA sweep** — for every endpoint with an object ID, replay across two accounts and as
   unauth/low-priv. This is the single most productive API test.
2. **BFLA sweep** — hit admin/management functions with a normal user token; change HTTP method
   (`GET`→`PUT`/`DELETE`/`PATCH`); try `/v1` of a `/v2`-protected action.
3. **Excessive data exposure** — read full JSON responses, not the rendered UI; APIs frequently return
   `passwordHash`, `ssn`, internal flags, other users' fields. Diff what the UI shows vs what the API
   sends.
4. **Mass assignment** — add privileged properties to write requests (`{"role":"admin"}`,
   `{"verified":true}`, `{"ownerId":<victim>}`).
5. **Rate/resource limits** — test login/OTP/reset and expensive endpoints for throttling; large
   `page_size`/`limit`; **GraphQL** deeply-nested and **batched/aliased** queries for amplification.
6. **Auth on every route** — strip the token; use an expired/another-tenant token; check JWT handling
   (alg confusion, no signature check — see [../identity/README.md](../identity/README.md)).

## GraphQL specifics

- **Introspection** on in prod → free schema map (turn it off in prod, or accept it's public).
- **Batching/aliasing** to bypass rate limits and amplify (e.g. 1000 aliased `login` mutations in one
  request → brute force / DoS). **Field-level authorization** is often missing — test each
  field/resolver, not just the top query. Depth/complexity limits frequently absent → DoS.
- Injection still applies inside resolver arguments → [../web/injection.md](../web/injection.md).

## gRPC / REST notes

- gRPC: enumerate via reflection; same authZ/validation concerns; check transport (mTLS) and that
  authorization is enforced in the service, not assumed at the gateway.
- REST: watch for verb-based authZ gaps and content-type confusion.

## DO NOT report as an API vulnerability if…

- The BOLA candidate endpoint **scopes the query by the authenticated user** (e.g.,
  `WHERE id = ? AND org_id = session.org`) — the ownership check is in the query itself
- The admin function **has role middleware you didn't see** (check decorators, guards,
  or a global route-level policy)
- The "excessive data" is in a **response only the owning user ever receives** (a user
  reading their own full profile is not data exposure)
- The rate-limit "bypass" is on a **read-only public endpoint** with no abuse impact
- You cannot demonstrate a **concrete unauthorized action** (no actual cross-account
  access, no actual priv-esc) — a *theoretical* missing check without proof belongs in
  "Needs Validation"

---

## Vulnerable ↔ fixed code examples

### BOLA (the #1 API bug — Broken Object Level Authorization)

```javascript
// ❌ VULNERABLE — fetches any order by ID, no ownership check
app.get('/api/orders/:id', authenticate, async (req, res) => {
  const order = await Order.findById(req.params.id);
  res.json(order);   // user A can read user B's order by guessing/iterating IDs
});

// ✅ FIXED — ownership scoped into the query
app.get('/api/orders/:id', authenticate, async (req, res) => {
  const order = await Order.findOne({
    _id: req.params.id,
    userId: req.user.id     // only returns if caller owns it
  });
  if (!order) return res.sendStatus(404);  // 404 not 403 (don't leak existence)
  res.json(order);
});
```

### BFLA (Broken Function Level Authorization)

```python
# ❌ VULNERABLE — admin endpoint with no role check
@app.route('/api/admin/users', methods=['DELETE'])
@login_required
def delete_user():
    user_id = request.json['user_id']
    User.query.get(user_id).delete()   # any authenticated user can delete anyone
    db.session.commit()

# ✅ FIXED — enforce admin role
@app.route('/api/admin/users', methods=['DELETE'])
@login_required
@require_role('admin')                  # middleware rejects non-admins with 403
def delete_user():
    user_id = request.json['user_id']
    User.query.get_or_404(user_id).delete()
    db.session.commit()
```

### Mass assignment (API3)

```javascript
// ❌ VULNERABLE — binds entire request body to model update
app.put('/api/profile', authenticate, async (req, res) => {
  await User.findByIdAndUpdate(req.user.id, req.body);
  // attacker sends {"role":"admin","verified":true} → instant priv-esc
});

// ✅ FIXED — allowlist of updatable fields
app.put('/api/profile', authenticate, async (req, res) => {
  const allowed = ['name', 'email', 'avatar'];
  const updates = Object.fromEntries(
    Object.entries(req.body).filter(([k]) => allowed.includes(k))
  );
  await User.findByIdAndUpdate(req.user.id, updates);
});
```

### Excessive data exposure (API3)

```python
# ❌ VULNERABLE — returns entire user object including internal fields
@app.route('/api/users/<int:user_id>')
@login_required
def get_user(user_id):
    user = User.query.get_or_404(user_id)
    return jsonify(user.to_dict())  # exposes password_hash, ssn, internal_notes

# ✅ FIXED — explicit response DTO with only allowed fields
@app.route('/api/users/<int:user_id>')
@login_required
def get_user(user_id):
    user = User.query.get_or_404(user_id)
    return jsonify({
        'id': user.id,
        'name': user.name,
        'avatar': user.avatar_url
    })
```

---

## Static signs (code audit)

```bash
rg -n "router\.(get|post|put|delete|patch)\(|@(Get|Post|Put|Delete)Mapping|app\.route" .   # endpoint inventory
rg -n "findById|findOne|\.get\(.*params\.id" .            # BOLA candidates (ownership scoped?)
rg -n "return.*user\b|res\.json\(user\)|serialize.*all|__dict__|model_to_dict" .  # over-exposure
```
Then verify each endpoint: authenticated? authorized to *this object*? returns only allowed fields?
rate-limited?

## CTF angle

API challenges: BOLA on `/api/users/{id}` or `/orders/{id}`, GraphQL introspection → hidden
`flag`/`admin` query, mass-assignment to `isAdmin`, or an old `/v1` endpoint missing the auth the `/v2`
has. Pull the schema, then hammer object/function authorization.

## Real-world cases

Disclosed HackerOne reports with PoCs:
[API](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPAPI.md) ·
[GraphQL](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPGRAPHQL.md).
Most are BOLA/BFLA + excessive data exposure — exactly the high-yield moves above.

## References

[OWASP API Security Top 10:2023](https://owasp.org/API-Security/editions/2023/en/0x00-header/) ·
[WSTG-APIT](https://owasp.org/www-project-web-security-testing-guide/) ·
OWASP [GraphQL](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html) &
[REST Security](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html) Cheat Sheets ·
CWE-[639](https://cwe.mitre.org/data/definitions/639.html)/[285](https://cwe.mitre.org/data/definitions/285.html)/[213](https://cwe.mitre.org/data/definitions/213.html).
Full bibliography: [research/api.md](../../research/api.md). Back to [tree](../00-map.md).
