# Broken Access Control (A01:2025)

**The #1 risk, and the #1 bug in vibe-coded CRUD apps.** Authentication asks "who are you?"; access
control asks "are you allowed to do *this* to *that*?" AI-generated code almost always wires up the
former and forgets the latter — the endpoint exists, the query runs, nobody checks ownership. CWE-639,
CWE-284, CWE-862/863, CWE-22.

## Sub-classes

- **IDOR / BOLA** (object-level) — you can read/modify *another user's object* by changing an ID.
- **Vertical priv-esc** (function-level / BFLA) — a low-priv user reaches an admin function.
- **Horizontal priv-esc** — user A acts as user B at the same privilege level.
- **Forced browsing** — unlinked but unprotected pages/endpoints (`/admin`, `/internal`).
- **Path traversal** — `../` escapes a directory to read arbitrary files (overlaps
  [file-upload-and-path.md](file-upload-and-path.md)).
- **Mass assignment** — binding user input to fields they shouldn't set (`isAdmin`, `role`, `userId`).
- **Missing method/segment checks** — `GET` is protected but `PUT`/`DELETE` isn't; `/v2` is, `/v1` isn't.

## Where it hides

Any endpoint that takes an **object identifier** (`/orders/123`, `/users/42/profile`, `?account=5`,
GraphQL `node(id:)`, a UUID, a filename, a slug) or any **privileged action** (admin panels, user
management, billing, export, impersonation). Also: indirect references in cookies/JWT claims/hidden
fields that the server trusts.

## How to find it

**Dynamic — the two-account method (most reliable):**
1. Create/obtain two accounts (A and B) and at least one low-priv + one admin if roles exist.
2. As A, exercise every action and record each request touching an object (IDs, UUIDs, filenames).
3. **Replay A's requests with B's session** (and vice-versa). If you get A's data while authenticated
   as B → **IDOR/BOLA**.
4. **Replay with no session / a low-priv session** against admin/privileged endpoints → **BFLA / vertical
   priv-esc**.
5. Toggle the **HTTP method** and **API version**; strip the auth header; swap a UUID for a sequential
   ID; try `id`, `user_id`, `account`, `X-User-Id` header overrides.
6. **Mass assignment:** add fields to the JSON body the UI doesn't send (`"role":"admin"`,
   `"isVerified":true`, `"userId":<victim>`) and see if they stick.

Burp's **Autorize** / **AuthMatrix** automate "replay this whole session as a lower-privilege user and
flag where access is still granted." Use them after a manual pass.

**Static — in code:** find every data-access call and ask *whose data does this return, and is
ownership checked?*
```bash
rg -n "findById|findOne|getById|\.get\(req\.params|where.*id\s*=" .
```
The classic vulnerable pattern:
```js
// ❌ no ownership check — any logged-in user reads any order
const order = await Order.findById(req.params.id);
res.json(order);
```
Also grep for object-binding that includes privileged fields (`Object.assign(user, req.body)`,
`User(**request.json)`, `Model.update(req.body)`), and admin routes lacking a role guard.

## How to confirm (PoC)

- IDOR: request object you don't own with your own session → receive another user's data. Prove scale
  by incrementing the ID a couple of steps (don't bulk-exfiltrate — a 2–3 record proof is enough; see
  the gate).
- BFLA: low-priv/no-auth call to an admin endpoint succeeds (returns data or performs the action).
- Mass assignment: response/DB reflects the privileged field you injected.

Capture exact request, the marker proving it's not yours, and the unauthorized result.

## How to fix

- **Deny by default; enforce server-side.** Every request re-checks: *is this principal allowed this
  action on this resource?* Never trust client-supplied role/ID/flags.
- **Scope every query to the principal:** `findOne({ id, ownerId: session.userId })` — not
  `findById(id)`. This single pattern kills most IDOR.
- **Centralize authorization** (policy/middleware/guard) instead of per-handler ad-hoc checks; use a
  policy engine (CASL, Oso, Casbin, Pundit) for non-trivial models.
- **Use unpredictable identifiers** (UUIDv4) as defense-in-depth — *not* as the control (still check
  ownership).
- **Allowlist bindable fields** (DTOs / `pick()` / serializer `fields`) to stop mass assignment; never
  bind raw request bodies to models.
- Protect *all* methods/versions/segments of an action, and check authorization **after**
  authentication, on the server, for APIs too.

```js
// ✅ ownership enforced in the query
const order = await Order.findOne({ id: req.params.id, ownerId: req.user.id });
if (!order) return res.sendStatus(404);   // 404 (not 403) to avoid leaking existence
```

## Advanced IDOR / BOLA patterns (where the easy checks pass)

- **Unpredictable IDs aren't a control:** UUIDs/GUIDs leak elsewhere (lists, exports, emails,
  `Location` headers, error messages); UUIDv1 is time/MAC-predictable; objects often also accept a
  guessable alias/slug alongside the UUID.
- **Secondary & nested objects:** the *primary* ID is authorized but a nested one isn't —
  `{"orderId":<mine>,"couponId":<victim>}`, `?tab=…&reportId=<victim>`. **Blind IDOR:** the action
  succeeds (state change) with no data echoed.
- **GraphQL global IDs:** `node(id:"<base64>")` where the id decodes to `Type:123` — decode, swap the id/type, re-encode;
  batch many IDs/aliases in one query to mass-extract (see [../api/README.md](../api/README.md)).
- **Method / version / shape confusion:** `GET`-protected but `PUT`/`PATCH` isn't; `/v2` guarded, `/v1`
  not; `id[]=` array vs scalar, JSON vs form body, and header overrides (`X-User-Id`, `X-Account`) the
  server trusts. Re-test every variant across two accounts and as unauth.

## CTF angle

Web CTF "auth" challenges are usually IDOR (swap the cookie/`user` param), forced browsing to a hidden
admin route, JWT `role` tampering ([../cryptography/README.md](../cryptography/README.md) for `alg:none`),
or mass assignment to set `admin=1`. Always try editing every client-controlled identity/role hint.

## Real-world cases

Disclosed HackerOne reports with PoCs, by bug type:
[IDOR](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPIDOR.md) ·
[broken authorization](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPAUTHORIZATION.md).
IDOR/BOLA is consistently the top-paid, highest-volume class in public programs.

## References

[OWASP A01:2025](https://owasp.org/Top10/2025/) · [WSTG-ATHZ](https://owasp.org/www-project-web-security-testing-guide/) ·
API1/API5:2023 (BOLA/BFLA — [../api/README.md](../api/README.md)) · CWE-[639](https://cwe.mitre.org/data/definitions/639.html)/[284](https://cwe.mitre.org/data/definitions/284.html)/[862](https://cwe.mitre.org/data/definitions/862.html) ·
OWASP [Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) & [IDOR Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html) Cheat Sheets ·
[PortSwigger: Access control](https://portswigger.net/web-security/access-control). Full bibliography: [research/web.md](../../research/web.md). Back to [web/](README.md).
