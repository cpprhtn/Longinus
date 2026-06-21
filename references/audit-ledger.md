# 📒 The Audit Ledger — coverage (recall) + the red×blue diff, as data

Most audits report a *list of bugs*. They can never answer the question that decides whether the audit was
any good: **"what did you look at and clear?"** (recall). The Audit Ledger is the data structure that makes
it answerable — and it is the surface the two lenses meet on:

- **Coverage / recall** ([the surface ledger](#1-surface--the-sourcesink-ledger-recall)) — every
  source→sink enumerated and given a verdict, so *un-examined* surface is visible instead of silent.
- **Red × Blue** ([red-blue.md](red-blue.md)) — the attacker fills `surface`, the defender fills
  `controls`, and **a finding is the diff between them**. The ledger is where the two lenses meet as data,
  not prose.

The ledger lives in the **report's YAML header** ([report-template.md](report-template.md)) — it is part
of the report, not a separate tool artifact. The full `surface[]`/`controls[]` arrays are emitted **only
when the audit builds one** (the multi-agent path, where Blue builds `controls[]`, or `deep` mode); the
single-skill/`standard` baseline carries the always-required `coverage:` counts and accounts for sinks in
§7 prose instead. The recall discipline below is the same either way.

> **Canonical schema.** This file is the single source of truth for the ledger schema; the YAML header in
> [report-template.md](report-template.md) mirrors it — if they disagree, this file wins. (Same
> keep-in-sync convention as [severity-and-triage.md](severity-and-triage.md) for FP discipline.)

---

## Why a ledger (recall is the missing half)

Longinus is ruthless about **precision** — *prove it or park it* is the crown jewel
([severity-and-triage.md](severity-and-triage.md)). But a security audit's most dangerous failure is not
a false positive (annoying); it is the **false negative** — the sink you never looked at. Precision and
recall are **orthogonal axes**, and the ledger is the recall instrument that the project previously
lacked.

> **The ledger never lowers the confirmation bar.** Coverage means *accounting for* each sink
> (`clean` / `finding` / `not-reached` / `not-examined`), **not reporting** each. A `surface` row marked
> `clean` is still subject to prove-it-or-park-it before anything becomes a `finding`. Recall = how much
> you examined; precision = how sure you are of what you filed. Raising one must not bend the other.

**Coverage metric:** `examined / total_sinks` — and the report must *name* the `not-examined` rows in
"What was NOT tested." A clean report with 40% coverage is an honest 40%, not a false "all clear."
If the target clearly has request handlers, parsers, privileged operations, or other attackable sinks
but `total_sinks = 0`, enumeration failed. Rerun with a scoped grep/static oracle or report the
enumeration failure as not tested; a zero denominator is never a clean bill.

---

## The schema (two tables in the report header)

The ledger has two tables. The LLM fills them as it audits; on the multi-agent path or `deep` mode they
ride in the report's YAML header so a human can inspect them and reports stay comparable (the
single-skill/`standard` baseline distills them to the `coverage:` counts + §7 prose — see
[report-template.md](report-template.md) rule 7).

### 1. `surface` — the source→sink ledger (recall)

One row per dangerous data flow. **Populated by the surface sweep** — a static-analysis oracle
(Semgrep taint, CodeQL, Joern, or a language LSP) when available, else by the entry-point/sink greps in
[audit-modes.md](audit-modes.md). The oracle is the **recall + reachability** engine; the LLM reasons on
top of it.

```yaml
surface:
  - id: S1
    source: "POST /api/order  body.total"      # where untrusted input enters
    sink: "services/pay.py:88  charge(amount)"  # where it becomes dangerous
    boundary: client→server                     # trust boundary crossed
    reachable: true                             # oracle/taint result (true|false|unknown)
    verdict: finding                            # clean | finding | not-reached | not-examined
    finding: F2                                 # → links to the finding, if any
```

- `reachable: unknown` is honest and common in static mode — it routes the row to **Needs-validation**,
  never to a confident finding (the *sink-without-source* trap in
  [severity-and-triage.md](severity-and-triage.md)).
- `verdict: not-examined` is the recall signal — un-swept sinks **and deferred multi-domain leaves**
  ([audit-modes.md](audit-modes.md) Step 4); the report must disclose them as gaps, never imply "clean."

### 2. `controls` — the expected-control map (Blue)

One row per control the **design** says should guard a boundary. **Populated by Blue** from the Intent
Brief ([design-intent.md](design-intent.md)) — *before* attacking. This is what turns "two lenses, one
flaw" from a sequential footnote into an actual diff.

```yaml
controls:
  - id: C1
    boundary: client→server                     # the boundary it guards
    expected: "server recomputes price from item IDs"  # the control intent says must exist
    present: false                              # is it actually implemented? (true|false|partial)
    bypassed: n/a                               # if present, can it be bypassed? (true|false|n/a)
    evidence: "no server-side recompute in services/pay.py"
```

### The diff = the finding (where red meets blue)

A finding is **mechanically derivable**, not vibes:

> **`surface.reachable == true` AND the guarding `controls` row is `present:false` or `bypassed:true`
> ⇒ a finding.** Conversely `reachable:true` + `present:true, bypassed:false` ⇒ `clean`. A
> `controls` row Blue expected but Red found *no matching surface for* ⇒ either dead defense or
> **un-enumerated surface** (recall gap — go enumerate).

Each `findings[]` entry (the existing report finding) carries `surface_id` and `control_id` so the
chain *source → sink → missing-control* is explicit and auditable.

---

## How each discipline uses the ledger

| Discipline | Reads | Writes | Produces |
|---|---|---|---|
| **Surface sweep** ([audit-modes.md](audit-modes.md)) | code, SA oracle | `surface[]` | the recall denominator |
| **Blue** ([red-blue.md](red-blue.md)) | Intent Brief | `controls[]` | expected-control map |
| **Red** ([pattern-triggers.md](pattern-triggers.md)) | `surface[]` | `surface.verdict`, `findings[]` | confirmed exploits |
| **Triage** ([severity-and-triage.md](severity-and-triage.md)) | the diff | `findings[].severity` | ranked, de-duped findings |
| **Continuous** ([continuous-audit.md](continuous-audit.md)) | the prior report | the delta | new / fixed / regressed |

## Graceful degradation (the ledger is scaffolding, not a gate)

No SA engine installed? Enumerate `surface` by grep and set `reachable: unknown` — coverage is still
counted, just lower-confidence. **The markdown methodology stands alone; the oracle only sharpens it** —
exactly like the SCA tools in [audit-modes.md](audit-modes.md).

## References

The schema is mirrored by [report-template.md](report-template.md) (the report header carries it on the
multi-agent/`deep` path; the baseline carries the `coverage:` counts — rule 7); the
recall philosophy pairs with the precision firewall in [severity-and-triage.md](severity-and-triage.md);
the two-lens diff is operationalized in [red-blue.md](red-blue.md). Full bibliography:
[research/process.md](../research/process.md).

Back to the [tree](00-map.md).
