# ⚔️ Red × Blue — the bidirectional method (make "two lenses" actually adversarial)

Longinus is built on *two lenses, one flaw*: the **attacker lens** (where it breaks) and the **defender
lens** (what control must be there). The weak way to run this is sequentially — hunt bugs, then bolt a
"fix" onto each. That is one lens with a footnote. The strong way is **adversarial**: two lenses that are
allowed to **disagree**, and the *disagreement itself is the finding*.

This file is the protocol that makes the method genuinely bidirectional. It runs on the shared
[Audit Ledger](audit-ledger.md): **Blue declares the controls that should exist; Red proves which sinks
are reachable; the flaw is the diff.**

> **Offense drives, defense seals — but defense goes *first* to aim offense.** Blue's control map is
> cheap (it falls out of the design intent you already read) and it *prunes* Red's work: Red attacks
> where a control *should* be but a reachable sink says it isn't.

---

## The four moves

```
   ┌─ BLUE: expected-control map ─┐        ┌─ RED: reachable-sink map ─┐
   │  (from design intent)        │        │  (from the surface sweep) │
   └──────────────┬───────────────┘        └──────────────┬────────────┘
                  │                                        │
                  └──────────────► DIFF ◄──────────────────┘
                         reachable sink ∧ missing/bypassed control
                                        │
                                        ▼
                              FIX → BYPASS loop (adversarial)
                       Blue proposes a control · Red tries to defeat it
                              until Red can't → the seal holds
```

### 1. Blue — build the expected-control map (before attacking)

From the **Intent Brief** ([design-intent.md](design-intent.md)), for **every designed trust boundary**,
write down the control that *should* guard it, and whether it is actually `present` / `bypassed`. This
fills `controls[]` in the [ledger](audit-ledger.md). Blue thinks like the *designer who is being honest
with themselves*: "this boundary needs server-side authorization — is it really there, on **this** path?"

Blue's sources of "what control should exist": the [enforce-forward.md](enforce-forward.md) per-category
control matrix (the canonical "what control kills this class") + the project's own stated intent.

### 2. Red — build the reachable-sink map

Independently, run the **surface sweep** ([audit-modes.md](audit-modes.md)) and the attacker lens
([pattern-triggers.md](pattern-triggers.md)) to enumerate `surface[]`: every source→sink, with
`reachable` from the static-analysis oracle. Red thinks like the attacker: "where does untrusted input
actually land, and can I get there?" Red does **not** read Blue's conclusions first — independence is
what kills confirmation bias (the same reason the [agents](../agents/README.md) isolate contexts).

### 3. Diff — the flaw is where they disagree

Overlay the two maps on the ledger ([the diff rule](audit-ledger.md#the-diff--the-finding-where-red-meets-blue)):

| Blue says | Red says | Verdict |
|---|---|---|
| control `present:true, bypassed:false` | sink `reachable:true` | **clean** — the seal holds |
| control `present:false` **or** `bypassed:true` | sink `reachable:true` | **🔴 finding** — reachable sink, no real control |
| control expected | **no matching surface** | recall gap *or* dead defense — go enumerate, don't assume safe |
| no control expected | sink `reachable:true` | **undocumented boundary** — still a finding ([design-intent](design-intent.md): silence ≠ defense) |

The richest findings are the **silent divergences**: the design says a control is there, Blue believed
it, and Red reaches the sink anyway. That sentence — *"intent says X is guarded; it is reachable
unguarded at `file:line`"* — is the deliverable a one-lens scanner can never write.

### 4. Fix → Bypass loop — adversarial sealing (not a one-shot patch)

A fix is not done when Blue proposes it; it is done when **Red cannot defeat it**. For each finding:

1. **Blue** proposes the structural control + CI/lint gate ([enforce-forward.md](enforce-forward.md)) —
   reported as **Current → Proposed → Trade-off**.
2. **Red** attacks the *proposed* control: the encoding variant the blocklist misses (`..%2f` past a
   `../` filter), the sibling endpoint, the second-order path, the parser differential
   ([pattern-triggers.md](pattern-triggers.md) principle 2). This is the
   [methodology.md](methodology.md) §9 retest, promoted to a first-class adversarial step.
3. If Red bypasses it → back to step 1 with a stronger control. If Red can't → the **seal holds**; record
   it. This is why an allowlist beats a blocklist, and a structural control beats a spot patch — they
   *survive* step 2.

---

## Mapping to the agent layer

The bidirectional method is structural in the [agents](../agents/README.md):

| Role | Agent | Writes |
|---|---|---|
| **Blue** | [longinus-blue](../agents/longinus-blue.md) — control-map builder | `controls[]` |
| **Red** | [longinus-web](../agents/longinus-web.md) / [-api-identity](../agents/longinus-api-identity.md) / [-cloud](../agents/longinus-cloud.md) / [-ai](../agents/longinus-ai.md) / [-secrets](../agents/longinus-secrets.md) | `surface[]`, `findings[]` |
| **Referee** | [longinus-orchestrator](../agents/longinus-orchestrator.md) — computes the diff, runs the fix→bypass loop | the report |

Blue runs **once, up front** (like the Intent Brief already does) and its control map is passed to every
Red specialist as shared context — so each attacks *with the expected defenses in hand* and reports the
gap, not its bug-class in a vacuum. Running standalone (no agents)? Do the four moves yourself in one
context — the discipline is identical, you just hold both maps in your head.

## Discipline (the firewall still holds on both sides)

- **Independence before the diff.** Don't let Blue's "there's surely auth here" suppress Red's probe, and
  don't let Red's bug-lust invent a missing control Blue can refute. Build both maps *before* reconciling.
- **Prove it or park it — still.** The diff *locates* a flaw; it does not confirm one. A `present:false`
  control over a `reachable:true` sink is a strong **lead** — confirm it with a PoC
  ([severity-and-triage.md](severity-and-triage.md)) before it leaves Needs-validation.
- **Cover it or flag it.** A `controls` row with no matching `surface` is not "safe" — it is
  *un-enumerated*. Recall gaps get disclosed, never assumed away ([audit-ledger.md](audit-ledger.md)).

## References

Blue's input is [design-intent.md](design-intent.md); Red's lens is
[pattern-triggers.md](pattern-triggers.md); the shared data structure is the
[Audit Ledger](audit-ledger.md); the seal Blue proposes is [enforce-forward.md](enforce-forward.md);
chained impact is [chaining-and-impact.md](chaining-and-impact.md). Full bibliography:
[research/process.md](../research/process.md).

Back to the [tree](00-map.md).
