# 🎯 Pattern triggers & attacker principles (unified lookup)

When auditing source code, use this file two ways:
- **Pattern-match (mechanical):** scan code, match a row in the tables below, open the
  referenced leaf for the full find→confirm→fix procedure.
- **Principle-driven (generative):** walk the six principles over the target; each one
  tells you *where it must break*, even for patterns not yet in the tables.

> **How to use:** scan the codebase (grep or read), match patterns below, open the
> referenced leaf. One pattern may appear multiple times across languages — check all
> that match the target stack. For patterns not in the table, reason from the six
> principles.

---

## 🧠 The six generative principles

A vulnerability is the gap between what the system was *designed* to do and what it
*actually* does with hostile input. These six principles *generate* all vulnerability
classes — including ones with no name yet.

### 1. Trust boundaries — untrusted data crosses into a context that trusts it

Every vulnerability is untrusted data crossing into a trusting context (SQL, shell, HTML,
URL fetch, tool call). Map where data enters and what trusts it downstream.

**Quick heuristic:** grep for every place user input is read (`req.body`, `req.params`,
`request.form`, `@RequestParam`). For each, trace forward: does it reach a sink (DB,
shell, HTML, file path, URL fetch, LLM prompt) without sanitization between? If yes →
open the matching sink's leaf.

### 2. Parser / impedance differentials — two components disagree on the same bytes

Bugs live where the *validator* and the *consumer* parse input differently — the
validation is theater, you smuggle past it.

**Quick heuristic:** look for input validated *before* being passed to a different
library that re-parses it. URL validation then `fetch()`; path validation then `open()`;
HTML sanitization then DOM insertion. If validator ≠ consumer → differential possible.

### 3. Confused deputy — a privileged component lends its authority to an attacker

A powerful component (server, browser, agent, cloud role) acts on input from a weak
caller without re-checking whose authority applies.

**Quick heuristic:** find every outbound request (`fetch`, `requests.get`, `curl`), file
operation, or cloud API call. Can the *user* control the target/arguments? If yes →
confused deputy. Also: any `PassRole`/`AssumeRole`/`actAs` — who can trigger it?

### 4. State, time & order — break the atomicity or sequence assumption

Systems assume operations are atomic and happen in the intended order. Do it twice at
once, out of order, skip a step, replay an old token.

**Quick heuristic:** grep for `if.*balance|if.*count|if.*used|if.*stock|if.*quantity`.
Is the check AND the modification in the **same DB transaction with a row lock**? If
separate (read → logic → write) → race candidate. Also: multi-step flows — can step N
be reached without completing step N-1?

### 5. Encoding layers — the value you check is not the value that runs

Validated input mutates before reaching the sink via canonicalization, decode round-trips,
or second-order storage. You validate one form; a different form executes.

**Quick heuristic:** search for `| safe`, `mark_safe`, `dangerouslySetInnerHTML`,
`v-html`, `noescape` — these disable auto-escaping. Also: data stored in DB then later
rendered without escaping (stored XSS); any blocklist-based filter (they almost always
miss an encoding variant).

### 6. The developer's "impossible" — unstated assumptions the developer never defends

"The client can't change this," "nobody sends a negative number," "you can't reach this
endpoint." Every assumption is an attack surface.

**Quick heuristic:** search for client-sent values that affect money, access, or state:
`req.body.price`, `req.body.role`, `req.body.isAdmin`. Also: scan the route table for
endpoints with no auth middleware — is each one genuinely public?

### Applying the principles

1. At **Profile / threat-model**, walk all six over the target. Most light up for any
   real app.
2. Each lit principle names the patterns to look for — match them in the tables below.
3. Found something? Push it through
   [chaining-and-impact.md](chaining-and-impact.md) — a principle often reveals the
   *next* boundary to cross.
4. A principle tells you *where to look*, not that a bug *exists* — still **prove it or
   park it** ([severity-and-triage.md](severity-and-triage.md)).

---

## ⛔ DO NOT report as a vulnerability if...

Before filing a finding, verify the pattern is actually exploitable. These guards
prevent the most common false positives:

1. **The input is already validated/parameterized upstream** — a parameterized query
   (`db.execute("... WHERE id = ?", (id,))`) is not SQLi even though it uses user input.
2. **The code is in a test file, mock, fixture, or documentation example** — not
   production-reachable.
3. **The code is dead / unreachable** — behind a permanent feature flag that's off, in a
   commented-out block, or in a function nothing calls.
4. **The framework auto-protects** — React JSX auto-escapes (no XSS unless
   `dangerouslySetInnerHTML`), Django ORM parameterizes (no SQLi unless `.raw()`/`.extra()`),
   Go `html/template` auto-escapes.
5. **The object ID is validated against the session owner before use** — an
   ownership-scoped query (`WHERE id = ? AND user_id = ?`) is not IDOR.
6. **The sink is only reachable by admin/internal callers with no untrusted path** —
   internal tooling with no external exposure is not a vulnerability (but document the
   assumption).
7. **The "missing header" has no demonstrable exploit** — don't report a missing
   `X-Content-Type-Options` without showing a content-type sniffing attack that actually
   fires.

**Principle-based guards** — when a principle lights up but it's NOT a bug:

| Principle | DO NOT report if... |
|---|---|
| 1. Trust boundary crossed | The framework auto-sanitizes at that boundary (parameterized ORM, React JSX auto-escape, Go `html/template`) |
| 2. Parser differential | Both parsers normalize to the same result *before* the security check, or only one parser is in the path |
| 3. Confused deputy | The deputy re-checks authorization **as the caller** (not its own identity) before acting |
| 4. State/time/order | The operation is inside a DB transaction with appropriate isolation level + row lock, or uses an atomic conditional update |
| 5. Encoding mutation | Input is canonicalized *before* the security check, and the check uses the same encoding form as the sink |
| 6. Developer's "impossible" | The assumption IS defended — just not where you looked (middleware, decorator, framework convention, global filter) |

**The meta-guard:** if you cannot construct a concrete input that demonstrates the
vulnerability, move it to **"Needs Validation"** rather than reporting it as confirmed.
See [limitations.md](limitations.md) for what static analysis fundamentally cannot reach.

When in doubt: state *"This pattern could be vulnerable to [X] if [precondition].
Verify that [specific defense] is in place."* — this is the **Needs Validation** bucket
from [severity-and-triage.md](severity-and-triage.md).

---

## Web / API patterns

| Code pattern you see | Check for | Leaf | False-positive guard |
|---|---|---|---|
| `Model.findById(req.params.id)` / `.get(pk=id)` without ownership filter | **IDOR / BOLA** | [web/access-control.md](web/access-control.md) | OK if middleware enforces ownership before handler, or query scoped by `user_id` |
| `db.execute(f"...{user_input}...")` / `"SELECT " + param` / `.raw(user_input)` | **SQL injection** | [web/injection.md](web/injection.md) | OK if param is from a static allowlist, or query is parameterized (`?` / `%s`) |
| `os.system(cmd)` / `exec(user)` / `child_process.exec(cmd)` with user data | **Command injection** | [web/injection.md](web/injection.md) | OK if using `execFile` with arg array (no shell), or input is from a strict allowlist |
| `render(template_string_with_user_input)` / `Template(user_input)` / `eval()` | **SSTI / code injection** | [web/injection.md](web/injection.md) | OK if the user input is passed as a variable to a precompiled template, not as the template itself |
| `res.send(userInput)` / `innerHTML = data` / `document.write(data)` / `v-html` / `dangerouslySetInnerHTML` | **XSS** | [web/xss.md](web/xss.md) | OK if framework auto-encodes (React JSX default) or input is sanitized with DOMPurify/Bleach immediately before |
| `{{ variable \| safe }}` / `mark_safe(variable)` / `Markup(variable)` | **XSS** (escaped output re-enabled) | [web/xss.md](web/xss.md) | OK only if the variable is guaranteed to be safe HTML from a trusted source |
| `fetch(req.body.url)` / `requests.get(user_url)` / `http.get(param)` | **SSRF** | [web/ssrf.md](web/ssrf.md) | OK if URL validated against a strict protocol+host allowlist (not just blocklist) |
| `req.body.price` / `req.body.total` / `req.body.discount` used in payment logic | **Price tampering** | [web/business-logic.md](web/business-logic.md) | OK if value is recomputed server-side from item IDs / DB prices |
| `if (balance >= amount) ... balance -= amount` as two separate operations | **Race condition** | [web/business-logic.md](web/business-logic.md) | OK if inside a DB transaction with row lock (`SELECT ... FOR UPDATE`) or atomic update |
| `Object.assign(user, req.body)` / `User(**request.data)` / `Model.update(req.body)` | **Mass assignment** | [api/README.md](api/README.md) | OK if DTO/allowlist filters fields before assignment |
| `return user.to_dict()` / `res.json(user)` / `serialize(model)` returning full objects | **Excessive data exposure** | [api/README.md](api/README.md) | OK if the serializer explicitly excludes sensitive fields |
| `pickle.load` / `pickle.loads` / `yaml.load(data, Loader=Loader)` (non-safe) | **Insecure deserialization → RCE** | [web/deserialization.md](web/deserialization.md) | OK if input is from a trusted source the attacker cannot control |
| State-changing `GET` handler / form without CSRF token / `SameSite=None` | **CSRF** | [web/csrf.md](web/csrf.md) | OK if all state-changing operations require a non-cookie auth header (e.g., Bearer token in API) |
| `path.join(base, req.params.file)` / `open(user_path)` without canonicalization | **Path traversal** | [web/file-upload-and-path.md](web/file-upload-and-path.md) | OK if `realpath()` checked against base and result is within allowed directory |
| File upload without content-type validation or stored in webroot | **Unrestricted upload** | [web/file-upload-and-path.md](web/file-upload-and-path.md) | OK if stored outside webroot with random name AND content-type validated server-side |
| Deep merge / recursive `Object.assign` / `_.merge` / `_.defaultsDeep` | **Prototype pollution** | [web/prototype-pollution.md](web/prototype-pollution.md) | OK if input is validated to reject `__proto__` / `constructor` / `prototype` keys |
| No auth middleware on a route definition / `@public` on a sensitive endpoint | **Broken authentication** | [web/auth-and-session.md](web/auth-and-session.md) | OK if the route genuinely requires no auth (public page, health check) |
| Admin/privileged route with same auth middleware as user routes | **BFLA** | [api/README.md](api/README.md) | OK if role/permission check exists inside or before the handler |

## Identity / Auth patterns

| Code pattern you see | Check for | Leaf | False-positive guard |
|---|---|---|---|
| `jwt.decode(token)` without `algorithms=[...]` pinned | **JWT alg confusion / alg:none** | [identity/README.md](identity/README.md) | OK if a separate verification step pins the algorithm |
| `jwt.decode(token, verify=False)` / `decode(token, options={"verify_signature": False})` | **JWT signature bypass** | [identity/README.md](identity/README.md) | OK only in a non-security context (logging, debugging) that never trusts the claims |
| `algorithms` list includes both `RS256` and `HS256` | **JWT RS→HS confusion** | [identity/README.md](identity/README.md) | Never OK — always pin to one algorithm family |
| `redirect_uri` compared with `.startsWith()` / `.includes()` / regex without `$` | **OAuth redirect_uri bypass → token theft** | [identity/README.md](identity/README.md) | OK only if exact-match allowlist is used |
| No `state` parameter in OAuth authorize/callback flow | **OAuth CSRF / login CSRF** | [identity/README.md](identity/README.md) | Never OK — state is mandatory |
| MFA check only in frontend JS / no server-side enforcement | **MFA bypass** | [identity/README.md](identity/README.md) | OK if server enforces MFA on every auth path |
| Password reset token with no expiry or reuse guard | **Account takeover via reset** | [web/auth-and-session.md](web/auth-and-session.md) | OK if token is single-use + time-limited + invalidated on use |

## AI / LLM patterns

| Code pattern you see | Check for | Leaf | False-positive guard |
|---|---|---|---|
| Model output inserted into `innerHTML` / SQL / shell / eval / URL fetch | **Improper output handling → XSS/SQLi/RCE/SSRF** | [ai-llm/prompt-injection.md](ai-llm/prompt-injection.md) | OK if output is escaped/parameterized/validated identically to raw user input |
| RAG vector search without tenant/user filter on query | **Cross-tenant data leakage** | [ai-llm/agentic-and-mcp.md](ai-llm/agentic-and-mcp.md) | OK if tenant filter is enforced in the vector store itself (namespace isolation) |
| Tool/function callable by model with no argument validation | **Excessive agency → argument injection** | [ai-llm/agentic-and-mcp.md](ai-llm/agentic-and-mcp.md) | OK if args are validated/allowlisted deterministically before execution |
| `trust_remote_code=True` in model loading | **Arbitrary code execution on load** | [ai-llm/model-supply-chain.md](ai-llm/model-supply-chain.md) | Never OK unless the repo is your own and pinned to a verified commit |
| `torch.load(file)` without `weights_only=True` | **Deserialization → RCE** | [ai-llm/model-supply-chain.md](ai-llm/model-supply-chain.md) | OK only if the file is produced internally and never touched by external parties |
| Sensitive tool (delete/pay/send) callable by agent without human confirmation | **Missing human-in-the-loop** | [ai-llm/agentic-and-mcp.md](ai-llm/agentic-and-mcp.md) | OK if the action is low-impact and reversible |
| System prompt containing API keys / DB passwords / secrets | **Secret in prompt → leakable** | [ai-llm/prompt-injection.md](ai-llm/prompt-injection.md) | Never OK — secrets must be in backend, not in prompts |

## Cloud / IaC patterns

| Code pattern you see | Check for | Leaf | False-positive guard |
|---|---|---|---|
| `Action: "*"` or `Resource: "*"` in IAM policy | **Over-privileged IAM** | [cloud-and-infra/cloud-iam.md](cloud-and-infra/cloud-iam.md) | OK if it's a `Deny` policy or scoped by a tight `Condition` |
| `acl = "public-read"` / `allUsers` / `allAuthenticatedUsers` in bucket/blob policy | **Public storage exposure** | [cloud-and-infra/cloud-iam.md](cloud-and-infra/cloud-iam.md) | OK only if the bucket intentionally serves public static assets with no sensitive data |
| `http_tokens = "optional"` or missing `metadata_options` block (AWS) | **IMDSv1 enabled → cred theft via SSRF** | [cloud-and-infra/cloud-iam.md](cloud-and-infra/cloud-iam.md) | Never OK — always require IMDSv2 |
| `USER root` or no `USER` directive in Dockerfile | **Container running as root** | [cloud-and-infra/iac-and-containers.md](cloud-and-infra/iac-and-containers.md) | OK if the image is used only for build/CI and never runs in production |
| `FROM image:latest` without digest pin | **Unpinned base image (supply chain)** | [cloud-and-infra/iac-and-containers.md](cloud-and-infra/iac-and-containers.md) | OK if pinned by `@sha256:...` digest elsewhere |
| `privileged: true` / `hostPath` / `hostNetwork` in k8s manifest | **Container escape risk** | [cloud-and-infra/iac-and-containers.md](cloud-and-infra/iac-and-containers.md) | OK only with a documented justification and strong compensating controls |
| `0.0.0.0/0` in security group / firewall ingress rule | **Unrestricted network exposure** | [cloud-and-infra/cloud-iam.md](cloud-and-infra/cloud-iam.md) | OK for load balancers / CDNs serving public traffic; not OK for SSH/DB/admin ports |
| Secrets in `.tf` / `tfvars` / `docker-compose.yml` / hardcoded env | **Hardcoded infrastructure secret** | [secrets-and-supply-chain/secret-detection.md](secrets-and-supply-chain/secret-detection.md) | Never OK — use a secret manager |

## Secrets / Supply chain patterns

| Code pattern you see | Check for | Leaf | False-positive guard |
|---|---|---|---|
| `api_key = "sk-..."` / `password = "..."` / `token = "ghp_..."` in source | **Hardcoded secret** | [secrets-and-supply-chain/secret-detection.md](secrets-and-supply-chain/secret-detection.md) | OK if it's a placeholder/example (e.g., `"your-api-key-here"` or in docs/tests with fake values) |
| No `package-lock.json` / `yarn.lock` / `poetry.lock` committed | **Unpinned dependencies** | [secrets-and-supply-chain/dependency-supply-chain.md](secrets-and-supply-chain/dependency-supply-chain.md) | OK if an alternative lockfile mechanism exists |
| Dependency with known CVE (check `npm audit` / `pip audit` / `trivy`) | **Vulnerable dependency** | [secrets-and-supply-chain/dependency-supply-chain.md](secrets-and-supply-chain/dependency-supply-chain.md) | Verify the CVE actually affects the used function/version, not just the package. **Unconfirmed reachability → Info / patch-anyway, not High/Critical** (see severity gates in [severity-and-triage.md](severity-and-triage.md)) |

---

When you encounter a pattern not in this table, reason from the six principles above —
that's how new bug classes get found.

Back to the [tree](00-map.md).
