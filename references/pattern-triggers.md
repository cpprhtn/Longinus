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
prevent the most common false positives. (Canonical FP discipline:
[severity-and-triage.md](severity-and-triage.md) "LLM-auditor failure modes" — these mirror it.)

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

## Code pattern → leaf lookup tables

The per-domain lookup tables (Web/API · Identity · AI/LLM · Cloud/IaC · Secrets · C/C++) — each with its
own false-positive-guard column — live in **[pattern-catalog.md](pattern-catalog.md)**. This file stays a
lean **principles + FP-guard** reference; pull the catalog in only when a code pattern's target leaf isn't
obvious. For patterns in no table, reason from the six principles above.

Back to the [tree](00-map.md).
