# 🧠 The attacker's mental model — how bug classes are *generated*

OWASP Top 10, the CWE list, this tree's leaves — those are **taxonomies**: maps for defenders and
report-writers. No good attacker memorizes 200 bug types. They carry **~6 generative principles** and
*derive* the bug on the spot, including classes that don't have a name yet. This doc is that lens.

> Read it once, then **bring it to every target** — it's not a checklist, it's the engine that
> produces checklists. The [00-map signal→file table](00-map.md) is the *symptom-driven* entry ("I see
> X, go to leaf Y"); this is the *principle-driven* entry ("I understand the system, where must it
> break?"). Use it during **Profile / threat-model** ([methodology.md](methodology.md) §1–3): walk the
> six lenses over the target; each one that lights up routes you to its leaves below.

The meta-move behind all six: **a vulnerability is the gap between what the system was *designed* to do
and what it *actually* does with hostile input.** Hold the spec and the implementation in your head at
once; attack the difference.

---

## 1. Map the trust boundaries first

Every vulnerability is **untrusted data crossing into a context that trusts it** — a SQL query, a
shell, a parser, an HTML renderer, a privileged identity, a tool call. Before you touch anything, draw
the boundaries: where does attacker-controlled data *enter*, and what *trusts it* downstream? The
entire tree is just "what happens at boundary X."

- **Tell:** follow the data; mark every place it crosses from "I control this" to "this is trusted."
- **Do this first** — it's the `input → sink → asset` sketch in [methodology.md](methodology.md) §1.
- **Instances:** essentially all of [web/](web/README.md); identity context →
  [identity/README.md](identity/README.md); the model/tool boundary →
  [ai-llm/agentic-and-mcp.md](ai-llm/agentic-and-mcp.md).
- **Quick heuristic (code audit):** grep for every place user input is read (`req.body`, `req.params`,
  `req.query`, `request.form`, `request.args`, `@RequestParam`, `params[:]`, `sys.argv`). For each,
  trace forward: does it reach a sink (DB, shell, HTML, file path, URL fetch, LLM prompt, IAM call)
  without sanitization/parameterization between? If yes → open the matching sink's leaf.

## 2. Hunt parser / impedance differentials

**Bugs live where two components disagree about the meaning of the same bytes.** If the *validator* and
the *consumer* parse input differently, the validation is theater — you smuggle past it. This is the
single most productive idea in offensive research of the last decade.

- **Tell:** two parsers anywhere in the path (front-end proxy + back-end; allowlist + the thing that
  actually fetches; browser DOM + server; one library's URL/JSON/XML parse vs another's), any
  decode/normalize layer, and the phrase *"we already validated that."*
- **Instances:** HTTP desync — front/back-end disagree on request length →
  [web/request-smuggling-and-desync.md](web/request-smuggling-and-desync.md); SSRF allowlist bypass via
  URL-parser confusion → [web/ssrf.md](web/ssrf.md); Unicode/charset normalization → mXSS →
  [web/xss.md](web/xss.md); object-merge semantics → [web/prototype-pollution.md](web/prototype-pollution.md);
  XML/SAML canonicalization → [identity/README.md](identity/README.md); content-type confusion →
  [web/file-upload-and-path.md](web/file-upload-and-path.md).
- **Quick heuristic (code audit):** look for any place where input is validated/sanitized
  *before* being passed to a different library/service that re-parses it. Especially: URL validation
  then `fetch()`; path validation then `open()`; HTML sanitization then DOM insertion; one JSON parser
  then another. If the validator and consumer are different implementations → differential possible.

## 3. Find the confused deputy

A **privileged component performs an action on behalf of a less-privileged caller without re-checking
whose authority should apply.** This is the generative core of SSRF, CSRF, SSO, and agentic tool abuse
— same bug, different deputy.

- **Tell:** "Component A (powerful — has a network position, a cookie, a token, a credential, root)
  acts on input from B (weak). Does it re-authorize the action as **B**, or does it lend B its own
  authority?"
- **Instances:** the server's network position → [web/ssrf.md](web/ssrf.md); the browser's ambient
  cookies → [web/csrf.md](web/csrf.md); token audience/`redirect_uri` →
  [identity/README.md](identity/README.md); the agent's credentials vs the end-user's →
  [ai-llm/agentic-and-mcp.md](ai-llm/agentic-and-mcp.md); an over-permissioned cloud role →
  [cloud-and-infra/cloud-iam.md](cloud-and-infra/cloud-iam.md).
- **Quick heuristic (code audit):** find every place the server makes an outbound request
  (`fetch`, `requests.get`, `http.get`, `curl`), performs a file operation, or calls a cloud API.
  For each: can the *user* control the target/arguments? If yes, the server is the confused deputy.
  Also: any `PassRole`/`AssumeRole`/`actAs` in cloud configs — who can trigger it?

## 4. Attack state, time & order

Systems assume operations are **atomic** and happen in the **intended sequence**. Break the assumption:
do it twice at once, out of order, skip a step, replay an old token.

- **Tell:** a check-then-act gap (validate, *then* use), anything counting (balance, quota, coupon,
  stock, retry counter), multi-step flows, and "this can only happen once."
- **Instances:** races / single-packet attack, step-skipping, double-spend →
  [web/business-logic.md](web/business-logic.md); session fixation, token replay →
  [web/auth-and-session.md](web/auth-and-session.md).
- **Quick heuristic (code audit):** grep for `if.*balance|if.*count|if.*used|if.*stock|if.*quantity`.
  For each: is the check AND the subsequent modification inside the **same DB transaction with a row
  lock**? If they're separate operations (read → app logic → write) → race condition candidate. Also:
  any multi-step flow (checkout, verify-email, password-reset) — can step N be reached without
  completing step N-1? Look for endpoints with no prior-state check.

## 5. Distrust every layer of encoding — *the value you check is not the value that runs*

Canonicalization, decode/re-encode round-trips, and second-order storage mean validated input often
**mutates before it reaches the sink**. You validate one form; a different form executes.

- **Tell:** data validated in one representation but used in another; data stored now and
  interpreted/executed later; any blocklist (it filters the form it can see, not the form that lands).
- **Instances:** second-order & blind injection → [web/injection.md](web/injection.md); stored/DOM XSS
  → [web/xss.md](web/xss.md); `..%252f` / double-decode path traversal, archive-extract path
  confusion → [web/file-upload-and-path.md](web/file-upload-and-path.md). This is also why **fix-bypass
  testing** matters ([methodology.md](methodology.md) §9): a filter that blocks `../` but not `..%2f`.
- **Quick heuristic (code audit):** search for `| safe`, `mark_safe`, `dangerouslySetInnerHTML`,
  `v-html`, `{!! !!}` (Blade), `noescape` — these explicitly disable auto-escaping. Each one must
  be justified by the developer. Also: data stored in DB then later rendered without escaping (stored
  XSS); and any blocklist-based filter (look for "deny" / "block" / "forbidden" lists — they almost
  always miss an encoding variant).

## 6. Hunt the developer's "impossible"

Every app rests on **unstated assumptions** the developer never defends: "the client can't change
this," "nobody sends a negative number," "this field is always small," "you can't reach this
endpoint," "the happy path is the only path." Each assumption is an attack surface.

- **Tell:** hidden form fields, client-side-only checks, sequential/guessable IDs, optimistic limits,
  the *unhappy* path (errors, cancellations, partial submits), and the route nobody linked to.
- **Instances:** IDOR/BOLA, forced browsing, mass assignment → [web/access-control.md](web/access-control.md)
  · [api/README.md](api/README.md); workflow/edge-case abuse → [web/business-logic.md](web/business-logic.md);
  exposed shadow/old endpoints → [api/README.md](api/README.md).
- **Quick heuristic (code audit):** search for any client-sent value that affects money, access, or
  state: `req.body.price`, `req.body.role`, `req.body.userId`, `req.body.isAdmin`,
  `req.body.discount`, `req.body.quantity`. Each is a "the client would never change this" assumption.
  Also: scan the route table for endpoints with no auth middleware — is each one genuinely public, or
  did someone forget?

---

## Applying the lens

1. At **Profile / threat-model**, walk all six over the target. Most lenses light up for any real app.
2. Each lit lens names the leaves to open — descend into **one**, run its find→confirm.
3. Found something? Don't stop at the bug — push it through
   [chaining-and-impact.md](chaining-and-impact.md). A lens often reveals the *next* boundary to cross.
4. Keep the discipline: a principle tells you *where to look*, not that a bug *exists* — still
   **prove it or park it** ([severity-and-triage.md](severity-and-triage.md)).

These six are the recurring structure underneath the named bug classes. When you meet a target — or a
technology — with no matching leaf, run the lenses anyway: that's how new bug classes get found.

## False-positive guards (when a principle lights up but it's NOT a bug)

Before reporting, verify the finding is real. A principle telling you *where to look* does not mean a
bug *exists* — still **prove it or park it**.

| Principle | DO NOT report if… |
|---|---|
| 1. Trust boundary crossed | The framework auto-sanitizes at that boundary (parameterized ORM, React JSX auto-escape, Go `html/template`) |
| 2. Parser differential | Both parsers normalize to the same result *before* the security check, or only one parser is in the path |
| 3. Confused deputy | The deputy re-checks authorization **as the caller** (not its own identity) before acting |
| 4. State/time/order | The operation is inside a DB transaction with appropriate isolation level + row lock, or uses an atomic conditional update |
| 5. Encoding mutation | Input is canonicalized *before* the security check, and the check uses the same encoding form as the sink |
| 6. Developer's "impossible" | The assumption IS defended — just not where you looked (middleware, decorator, framework convention, global filter) |

**The meta-guard:** if you cannot construct a concrete input (even a theoretical one showing the logic
flow) that demonstrates the vulnerability, move it to **"Needs Validation"** rather than reporting it
as confirmed. See [limitations.md](limitations.md) for what static analysis fundamentally cannot reach.

## References

The generative view of vulnerabilities draws on the **confused-deputy** concept (Norm Hardy, 1988),
**Saltzer & Schroeder** least-privilege/complete-mediation principles, and the parser-differential line
of research popularized by **PortSwigger Research / James Kettle** (HTTP desync) and **Orange Tsai**
(URL-parser confusion) — all catalogued in [research/process.md](../research/process.md) and the
per-domain bibliographies. Pairs with [methodology.md](methodology.md) (the lifecycle) and
[chaining-and-impact.md](chaining-and-impact.md) (turning a bug into impact).

Back to the [tree](00-map.md).
