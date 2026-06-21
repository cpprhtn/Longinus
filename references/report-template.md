# 📋 The Longinus report template — emit this **exactly**

Every **standard / deep** audit produces a report in **this exact structure** — same sections, order, and
per-finding fields, **every project and every run**. (*Quick* mode is the exception: it emits the
provisional single-table output from [audit-modes.md](audit-modes.md), then a `standard` re-run produces
this full report.) Two readers must both be served: a **human** scanning it (so it is visual and
scannable) and a **machine** aggregating it (so the YAML header is parseable). Reports must stay
**diffable and comparable** across a fleet of projects.

> **Rules (don't break these — they're why reports were inconsistent; full rationale in
> [reporting-and-disclosure.md](reporting-and-disclosure.md)):**
> **(1)** copy the skeleton **verbatim** — never rename/reorder/merge/drop a section (N/A → keep the heading,
> write `None`). **(2)** severity = **badge + the canonical English enum word** (🔴 `Critical` · 🟠 `High` ·
> 🟡 `Medium` · 🔵 `Low` · ⚪ `Informational` · ❓ `Needs-validation`) *even in a localized report* — the
> dashboard/`counts` aggregate on it. **(3)** the **dashboard** (overall-risk line + counts table) is
> mandatory. **(4)** findings get **stable F-IDs** (`F1…`, highest-severity first) + a one-line **Impact**;
> §3 references the same F-IDs. **(5)** **§4 = Confirmed only · §6 = Needs-validation** (never blur; fill
> every field) — **Status = the evidence tier** `✅ Confirmed (executed)` > `✅ Confirmed (traced)` >
> `❓ Needs-validation`, never "executed" for an unrun PoC ([proof-and-confirmation.md](proof-and-confirmation.md)).
> **(6)** the **`coverage:` counts are required every report**; the full `surface[]`/`controls[]` arrays ride
> in the header **only on multi-agent/`deep`** (single-skill/`standard` → account for sinks in §7 prose), and
> **§7 lists examined vs not-examined** ([audit-ledger.md](audit-ledger.md)). If a target clearly has
> attackable inputs/sinks but `coverage.total = 0`, the audit is **incomplete**: rerun enumeration or
> report the enumeration failure in §7; never emit a clean report from a zero denominator.
> **(7)** **match the request's
> language** for prose; keep the **YAML header, enum words, CWE/OWASP/CVSS ids, and `Status`/`Effort` labels
> canonical English**. **(8)** write to `.longinus/reports/longinus_YYYYMMDDHHMM.md`
> ([continuous-audit.md](continuous-audit.md)).

---

````markdown
---
longinus_report: "0.7.0"   # = the Longinus skill version (SKILL.md); not a separate schema number
target: <repo name or path>
commit: <short SHA, or n/a>
date_utc: <YYYY-MM-DD HH:MM>
mode: <quick|standard|deep|continuous>
scope: <one line — in/out of scope; static or dynamic>
overall_risk: <Critical|High|Medium|Low|Informational>
counts: { critical: 0, high: 0, medium: 0, low: 0, info: 0, needs_validation: 0 }
coverage: { examined: 0, total: 0, pct: 0 }   # examined sinks / enumerated sinks — see audit-ledger.md
# surface[] / controls[] ledger arrays — OPTIONAL: emitted here only on the multi-agent path or `deep`
# mode (where Blue builds controls[]); the single-skill/standard baseline omits them and accounts for
# sinks in §7 prose instead. Schema: audit-ledger.md
---

# 🗡️ Longinus security audit — <target>

> ### Overall risk: <🔴 Critical | 🟠 High | 🟡 Medium | 🔵 Low | ⚪ Informational>
> <one sentence — the single most important takeaway.>

| 🔴 Critical | 🟠 High | 🟡 Medium | 🔵 Low | ⚪ Info | ❓ Needs-val |
|:-:|:-:|:-:|:-:|:-:|:-:|
| 0 | 0 | 0 | 0 | 0 | 0 |

`commit <sha>` · `<mode>` mode · `<date_utc> UTC` · scope: <one line>
**Coverage:** examined <N>/<total> sinks (<pct>%) — un-examined surface is listed in §7. *(Coverage ≠ confidence: a low number is an honest gap, not a clean bill — [audit-ledger.md](audit-ledger.md).)*

## 1. Executive summary
<3–6 sentences: posture, findings by severity, top risk, headline fix.>

## 2. Design intent brief
- **Purpose:** <what it's for; crown jewels>
- **Trust boundaries (as designed):** <who/what is trusted where>
- **Stated assumptions:** <"client can't change X", "internal-only", …>
- **Documented accepted-risks:** <cited, or "none found">

## 3. Fix list
> Ordered by severity. *Effort* lets you sequence quick wins first; **when** to fix is your team's call,
> not the auditor's — the report rates risk, not your roadmap.

| F# | Severity | Finding | Effort |
|:-:|:-:|---|:-:|
| F1 | 🔵 Low | <title> | S |

*Effort: **S** = a line/config · **M** = one module/component · **L** = re-architecture. Severity = how bad it is; Effort = how cheap to fix.*

## 4. Findings
<one block per CONFIRMED finding, highest severity first>

### 🔵 F1 · Low — <Title that names the bug and its impact>
> **Impact:** <one-line TL;DR a busy reader can act on — concrete, and at what scale.>

| | |
|---|---|
| **Severity** | 🔵 Low · CVSS 4.0 <score / vector or n/a> |
| **Weakness** | CWE-<n> · OWASP <ref or n/a> |
| **Location** | `<file:line>` / `<endpoint>` |
| **Status** | ✅ Confirmed (executed) — *or* ✅ Confirmed (traced) |

- **Intent reconciliation:** <documented accepted-risk → Informational + cite | contradicts intent → raised | undocumented assumption → "confirm intended" | n/a>
- **PoC / reproduction:** <numbered, copy-pasteable, benign marker only; if *executed*, the exact command **and the observed output** that proves it>
- **Fix:**
  - *Current:* <what the code/config does now>
  - *Proposed:* <the change — prefer a diff>
  - *Trade-off:* <what applying the proposed change costs — performance or UX (latency, an extra step, a broken integration), or "none". Informational; never lowers severity.>
- **Enforcement gate:** <structural control + CI/lint gate that kills the class, or n/a>

---

### <🔴/🟠/🟡/🔵/⚪> F2 · <Severity> — <Title>
> **Impact:** …

<…same field layout…>

## 5. Dependency health (SCA)
<reachable CVEs vs patch-anyway items, separated — or "None" / "Not run (tool unavailable)">

## 6. Needs-validation
<unconfirmed items, clearly separated — NEVER mixed into §4. Or "None.">

## 7. What was NOT tested
<honest coverage gaps: dynamic behavior, classes N/A, tools missing, out-of-scope. **Enumerate what was
examined vs not** — when a `surface[]` ledger exists (multi-agent / `deep`), list the rows whose verdict is
`not-examined` / `reachable: unknown`; otherwise (single-skill/`standard` baseline) account for the sinks in
prose. These are the recall gaps ([audit-ledger.md](audit-ledger.md)). Coverage % above must reconcile with
this list.>

## 8. Next run (incremental)
Recorded commit `<sha>`. A re-run audits `git diff <sha>..HEAD` and appends a delta
(new / fixed / regressed). See continuous-audit.md.
````

---

The *why* behind each field (writing principles, disclosure rules) lives in
[reporting-and-disclosure.md](reporting-and-disclosure.md); the file-output + incremental contract is
[continuous-audit.md](continuous-audit.md); the Intent Brief in §2 comes from
[design-intent.md](design-intent.md); the `coverage` + `surface[]`/`controls[]` ledger schema is canonical
in [audit-ledger.md](audit-ledger.md). Full bibliography: [research/process.md](../research/process.md).
Back to [tree](00-map.md).
