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

## 4. Attack state, time & order

Systems assume operations are **atomic** and happen in the **intended sequence**. Break the assumption:
do it twice at once, out of order, skip a step, replay an old token.

- **Tell:** a check-then-act gap (validate, *then* use), anything counting (balance, quota, coupon,
  stock, retry counter), multi-step flows, and "this can only happen once."
- **Instances:** races / single-packet attack, step-skipping, double-spend →
  [web/business-logic.md](web/business-logic.md); session fixation, token replay →
  [web/auth-and-session.md](web/auth-and-session.md).

## 5. Distrust every layer of encoding — *the value you check is not the value that runs*

Canonicalization, decode/re-encode round-trips, and second-order storage mean validated input often
**mutates before it reaches the sink**. You validate one form; a different form executes.

- **Tell:** data validated in one representation but used in another; data stored now and
  interpreted/executed later; any blocklist (it filters the form it can see, not the form that lands).
- **Instances:** second-order & blind injection → [web/injection.md](web/injection.md); stored/DOM XSS
  → [web/xss.md](web/xss.md); `..%252f` / double-decode path traversal, archive-extract path
  confusion → [web/file-upload-and-path.md](web/file-upload-and-path.md). This is also why **fix-bypass
  testing** matters ([methodology.md](methodology.md) §9): a filter that blocks `../` but not `..%2f`.

## 6. Hunt the developer's "impossible"

Every app rests on **unstated assumptions** the developer never defends: "the client can't change
this," "nobody sends a negative number," "this field is always small," "you can't reach this
endpoint," "the happy path is the only path." Each assumption is an attack surface.

- **Tell:** hidden form fields, client-side-only checks, sequential/guessable IDs, optimistic limits,
  the *unhappy* path (errors, cancellations, partial submits), and the route nobody linked to.
- **Instances:** IDOR/BOLA, forced browsing, mass assignment → [web/access-control.md](web/access-control.md)
  · [api/README.md](api/README.md); workflow/edge-case abuse → [web/business-logic.md](web/business-logic.md);
  exposed shadow/old endpoints → [api/README.md](api/README.md).

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

## References

The generative view of vulnerabilities draws on the **confused-deputy** concept (Norm Hardy, 1988),
**Saltzer & Schroeder** least-privilege/complete-mediation principles, and the parser-differential line
of research popularized by **PortSwigger Research / James Kettle** (HTTP desync) and **Orange Tsai**
(URL-parser confusion) — all catalogued in [research/process.md](../research/process.md) and the
per-domain bibliographies. Pairs with [methodology.md](methodology.md) (the lifecycle) and
[chaining-and-impact.md](chaining-and-impact.md) (turning a bug into impact).

Back to the [tree](00-map.md).
