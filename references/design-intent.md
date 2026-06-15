# 🎯 Design-intent first — audit the gap, not the whole codebase

A vulnerability is the **gap between what the system was *designed* to do and what it *actually* does
with hostile input** — the meta-axiom behind [pattern-triggers.md](pattern-triggers.md). So **read the
design intent first**, then aim testing at the input vectors that *violate or stress* that intent,
instead of scanning everything blind. This is the academic, differentiated entry: most precise when the
intent is *explicit* (you target the gap directly) and most disciplined when reconciling findings
against *documented* decisions — so a mid-risk issue the designer left on purpose isn't mis-reported.

## Phase 0.5 — extract the Intent Brief (after the gate, during Profile)

Read, in order, whatever exists:

- `CLAUDE.md`, `README`, `docs/`, `ARCHITECTURE.md`, **ADRs** (`docs/adr/`, `*.adr.md` — architecture
  decision records), `SECURITY.md` / security policy, `THREATMODEL`, design/RFC docs, the API contract
  (OpenAPI/GraphQL schema/`.proto`), and **intent-bearing code comments** (`# only admins reach here`,
  `// trusted input`, `assumes X is pre-validated`).

Produce a short **Intent Brief**:

1. **Purpose & crown jewels** — what the system is for; what's valuable.
2. **Trust boundaries *as designed*** — who/what the designer treats as trusted vs untrusted, and where.
3. **Stated assumptions** — "the client can't change X", "this endpoint is internal-only", "input is
   validated upstream", "this runs only in CI".
4. **Documented accepted-risks** — risks the team explicitly took ("public bucket serves static assets,
   no PII — intentional", "rate-limiting handled at the CDN").

## Focus testing on intent-violating input vectors

Don't audit everything equally — aim where reality can diverge from intent:

- **Every stated assumption is an attack hypothesis.** "Internal-only" → can an external request reach
  it? "Client can't change X" → can it? "Validated upstream" → is it, on *this* path?
- **Every designed trust boundary** → test that the *implementation* actually enforces it (design says
  one thing; code may not).
- **The contract edges** (schema/OpenAPI) → inputs outside the declared shape, types, and ranges.

This is the attacker's "hunt the developer's *impossible*" — but with the impossible-list handed to you
in writing. The richest findings are where **the implementation silently diverges from the stated design.**

## Reconcile findings against intent — the discipline (NOT a bug-excuse)

| The finding, vs the intent… | Verdict |
|---|---|
| Matches a **documented** accepted-risk | **Downgrade to Informational** — *and cite the doc/comment.* This is the *only* legitimate "the designer meant it." |
| **Contradicts** stated intent | A real finding — often **higher** severity: the system does what it was *not* designed to do. |
| Relies on an **undocumented** assumption | **Still a finding.** Report it + *"confirm this is intended."* An unstated assumption is exactly what an attacker exploits — silence is not a defense. |

> **Intent gives *context*, never *absolution*.** A mid-risk issue counts as "deliberate" only if it is
> *written down*; otherwise it stays a finding. This keeps design-intent from becoming a way to wave
> bugs away — see the false-positive firewall in [severity-and-triage.md](severity-and-triage.md).

## In the multi-agent flow

The **orchestrator** builds the Intent Brief **once, up front**, and passes it to every specialist as
shared context — so each agent audits *with the design in hand* (it knows what's deliberate, where the
boundaries are, which assumptions to attack) instead of pursuing its bug-class in a vacuum. See
[../agents/README.md](../agents/README.md).

## When there's no design doc

Infer intent from structure (route tables, auth middleware, naming, types, tests), label the brief
**"inferred intent (low-confidence)"**, and proceed — but treat *every inferred assumption as
unverified*: that's **more** to test, not a license to assume safety.

## References

The "gap between design and behavior" framing underlies [pattern-triggers.md](pattern-triggers.md)
(principle 6) and the threat-model step of [methodology.md](methodology.md); reconciliation feeds
[severity-and-triage.md](severity-and-triage.md). Full bibliography:
[research/process.md](../research/process.md). Back to [tree](00-map.md).
