# 📋 The Longinus report template — emit this **exactly**

Every audit produces a report in **this exact structure** — same sections, order, and per-finding
fields, **every project and every run**. Two readers must both be served: a **human** scanning it (so it
is visual and scannable) and a **machine** aggregating it (so the YAML header is parseable). Reports must
stay **diffable and comparable** across a fleet of projects.

> **Rules (do not break these — they are why reports were inconsistent):**
> 1. **Copy the skeleton verbatim** — don't rename, reorder, merge, split, or "lighten" sections.
> 2. **Never drop a section** — if N/A, keep the heading and write `None` / `N/A`.
> 3. **Severity is always a badge + word:** 🔴 `Critical` · 🟠 `High` · 🟡 `Medium` · 🔵 `Low` ·
>    ⚪ `Informational` · ❓ `Needs-validation`. Use these exact words.
> 4. **Top dashboard is mandatory** (overall-risk line + the counts table) — it's the human at-a-glance.
> 5. **Findings get stable IDs** `F1, F2, …` (so re-runs track the same finding over time), ordered by
>    severity (highest first), each with a one-line **Impact** TL;DR.
> 6. **§4 Findings = Confirmed only. §6 = Needs-validation.** Never blur the two. Fill *every* field
>    (use `n/a`, never delete one).
> 7. Write the file to `.longinus/reports/longinus_<UTC>.md` ([continuous-audit.md](continuous-audit.md)).

---

````markdown
---
longinus_report: "1.1"
target: <repo name or path>
commit: <short SHA, or n/a>
date_utc: <YYYY-MM-DD HH:MM>
mode: <quick|standard|deep|continuous>
scope: <one line — in/out of scope; static or dynamic>
overall_risk: <Critical|High|Medium|Low|Informational>
counts: { critical: 0, high: 0, medium: 0, low: 0, info: 0, needs_validation: 0 }
---

# 🗡️ Longinus security audit — <target>

> ### Overall risk: <🔴 Critical | 🟠 High | 🟡 Medium | 🔵 Low | ⚪ Informational>
> <one sentence — the single most important takeaway.>

| 🔴 Critical | 🟠 High | 🟡 Medium | 🔵 Low | ⚪ Info | ❓ Needs-val |
|:-:|:-:|:-:|:-:|:-:|:-:|
| 0 | 0 | 0 | 0 | 0 | 0 |

`commit <sha>` · `<mode>` mode · `<date_utc> UTC` · scope: <one line>

## 1. Executive summary
<3–6 sentences: posture, findings by severity, top risk, headline fix.>

## 2. Design intent brief
- **Purpose:** <what it's for; crown jewels>
- **Trust boundaries (as designed):** <who/what is trusted where>
- **Stated assumptions:** <"client can't change X", "internal-only", …>
- **Documented accepted-risks:** <cited, or "none found">

## 3. Fix list (do these first)
| # | Severity | Finding | Priority |
|:-:|:-:|---|:-:|
| 1 | 🔵 Low | <title> | Soon |

## 4. Findings
<one block per CONFIRMED finding, highest severity first>

### 🔵 F1 · Low — <Title that names the bug and its impact>
> **Impact:** <one-line TL;DR a busy reader can act on — concrete, and at what scale.>

| | |
|---|---|
| **Severity** | 🔵 Low · CVSS 4.0 <score / vector or n/a> |
| **Weakness** | CWE-<n> · OWASP <ref or n/a> |
| **Location** | `<file:line>` / `<endpoint>` |
| **Status** | ✅ Confirmed |

- **Intent reconciliation:** <documented accepted-risk → Informational + cite | contradicts intent → raised | undocumented assumption → "confirm intended" | n/a>
- **PoC / reproduction:** <numbered, copy-pasteable, benign marker only>
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
<honest coverage gaps: dynamic behavior, classes N/A, tools missing, out-of-scope.>

## 8. Next run (incremental)
Recorded commit `<sha>`. A re-run audits `git diff <sha>..HEAD` and appends a delta
(new / fixed / regressed). See continuous-audit.md.
````

---

The *why* behind each field (writing principles, disclosure rules) lives in
[reporting-and-disclosure.md](reporting-and-disclosure.md); the file-output + incremental contract is
[continuous-audit.md](continuous-audit.md); the Intent Brief in §2 comes from
[design-intent.md](design-intent.md). Full bibliography: [research/process.md](../research/process.md).
Back to [tree](00-map.md).
