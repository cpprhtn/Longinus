# ⚠️ Known limitations — what this skill CANNOT find

Be honest about what a static/LLM-driven analysis misses. When these apply, state
**"not tested / not testable in this mode"** in the report — never imply "no
vulnerabilities found" when the truth is "this class wasn't reachable."

---

## Cannot find (static code audit mode)

| # | Limitation | Why |
|---|---|---|
| 1 | **Runtime-only race conditions** | True concurrency issues need dynamic testing (multiple simultaneous requests); static analysis can flag the *pattern* but cannot prove the window is exploitable |
| 2 | **Actual cloud/infra state** | Can only assess IaC/config *as written*; real drift (manually changed security groups, rotated secrets, disabled logging) is invisible |
| 3 | **Business logic requiring domain expertise** | "Is this business rule correct?" depends on product intent, not code structure — can flag suspicious patterns but cannot prove a rule is wrong without domain context |
| 4 | **Novel vulnerabilities in compiled dependencies** | Can flag *known* CVEs by version (SCA), but cannot find zero-days in library internals |
| 5 | **Complex client-side DOM XSS** | Tracing data flow through heavily minified/bundled JS without runtime execution is unreliable |
| 6 | **Cryptographic implementation flaws** | Can flag *known weak patterns* (ECB mode, short keys, predictable IV) but cannot perform actual cryptanalysis |
| 7 | **Side-channel attacks** | Timing, cache, power analysis — not detectable from source code alone |
| 8 | **Social engineering / phishing** | Out of scope entirely |
| 9 | **Reachability of a code path under production conditions** | Can find the vulnerable *code* but may not prove it's reachable from production input without runtime tracing. **Mitigations:** (a) run a static-analysis taint oracle (Semgrep/CodeQL/Joern) in the [surface sweep](audit-modes.md) — it raises reachability confidence and fills the [ledger](audit-ledger.md); (b) on owned/local code, **actually execute a benign PoC** ([proof-and-confirmation.md](proof-and-confirmation.md)) to settle it. What neither resolves is recorded as `reachable: unknown` / `Confirmed (traced)` (a disclosed gap), never a confident invented finding |
| 10 | **Complete coverage of a large codebase in one pass — recall is hostage to the context window** | The most dangerous failure is *silent*: on a big repo the model may actually load only part of the tree and — not knowing what it never read — **under-report its own §7 coverage gap**, so a *partial* audit looks *complete*. Precision discipline (prove-or-park) does not protect recall here. The `coverage.total == 0` guard only catches *total* failure, not partial. **Mitigations:** enumerate a **module/entrypoint inventory first** so `coverage.total` counts *modules*, not just found sinks; audit module-by-module (map-reduce) with an explicit **loaded-vs-skipped ledger** ([audit-ledger.md](audit-ledger.md)); when no taint oracle is available, declare the indirect-flow blind spot (DI containers, event emitters, ORM, reflection, dynamic dispatch) in §7 rather than implying it was covered |

## Reduced confidence (flag but caveat)

These *can* be found but with lower confidence — report them as **"Needs Validation"**:

| # | Area | Why confidence is lower |
|---|---|---|
| 1 | **Second-order / stored injection** | Data stored now, used unsafely later in a different code path — can flag the sink but may miss the connection from source to sink |
| 2 | **Complex authorization logic** | Deeply nested RBAC/ABAC with inheritance, wildcards, and exceptions — individual gaps may not be visible from single code paths |
| 3 | **Non-obvious multi-step chains** | Can identify individual links but may miss multi-step chains that require specific ordering or preconditions |
| 4 | **Framework "magic"** | ORMs, middleware, decorators, annotations may handle security transparently — may false-positive if the framework protects it invisibly |
| 5 | **Cross-file impact in diff-only (continuous) re-audits** | Auditing only the `git diff` ([continuous-audit.md](continuous-audit.md)) misses the case where a change in file A makes *existing, previously-safe* code in file B exploitable — e.g. a new caller now reaches an old sink, or a widened type flows into an unchanged query. **Mitigation:** expand scope to the **callers of every changed symbol**, not just the changed lines; when that isn't feasible, say so in §7 |

## Session contamination (multi-audit sessions)

When multiple audits run in the same conversation, prior findings contaminate the
model's analysis of subsequent targets:

| # | Effect | Manifestation |
|---|---|---|
| 1 | **Confirmation bias** | Findings from project A cause over-reporting of the same class in project B |
| 2 | **Severity anchoring** | High/Critical findings in the first audit inflate severity estimates for the second |
| 3 | **Pattern fixation** | The model over-indexes on patterns it already found, missing novel issues |
| 4 | **Context overflow** | Prior audit content fills the context window, reducing capacity for the current target |

**Mitigation:** run `/clear` between audits of different targets. Each audit should
start with a clean context. This is a fundamental limitation of session-based LLM
analysis, not a skill defect.

## Engine sensitivity (model capacity & determinism)

The auditor is an LLM, so two properties of the *engine* — not the target — bound what you can trust:

| # | Property | What it means for you |
|---|---|---|
| 1 | **Capacity sensitivity** | The heaviest-reasoning disciplines — chaining across boundaries, deriving bugs from the six principles, the severity gates — are the **first to degrade on smaller models**, and that is exactly where false-positive suppression lives. On small/on-device models (the `quick` audience), trust the mechanical scans and treat chaining / severity / novel-principle findings as **provisional**, re-run under a stronger model before acting. |
| 2 | **Non-determinism** | The same target audited twice can yield a **different finding set** (LLM sampling). A *flaky* CI gate gets disabled by the team within days. So `quick` / CI-gate mode must anchor on **fixed-ruleset tool output** (Semgrep + gitleaks + SCA) for enumeration and constrain the LLM to **triage / dedup only** — the mechanical core is what stays reproducible; the LLM layer is provisional by design ([audit-modes.md](audit-modes.md)). |

## DO NOT report as a finding

> Mirrors the canonical [severity-and-triage.md](severity-and-triage.md) "LLM-auditor failure modes" —
> keep in sync.


- Code in **test files, mocks, fixtures, or documentation examples** — not production
- **Dead code** — unreachable, behind a permanent-off feature flag, or in deleted/commented blocks
- Patterns protected by **framework defaults** (React auto-escaping, Django ORM
  parameterization, Go `html/template`, Rails `ActiveRecord` parameterized queries)
- **"Missing security header" without a demonstrable exploit** — `X-Content-Type-Options`
  alone without a content-sniffing attack vector is informational, not a vulnerability
- **Known-acceptable risks explicitly documented in the codebase** (e.g., a public
  bucket that intentionally serves static assets with a comment explaining why)
- **Version-only CVE** where the vulnerable function/code path is never called
- **Hallucinated CVE** — a CVE number you "remember" but cannot verify exists; if you
  cannot cite the exact CVE with a real link, do not report it

## When in doubt

Use this template:

> **Needs Validation:** This pattern *could* be vulnerable to [X] if [precondition Y is
> true]. Verify that [specific defense Z] is in place. If Z is confirmed, this is a
> false positive.

This goes in the "Needs Validation" bucket — visibly separate from confirmed findings
per [severity-and-triage.md](severity-and-triage.md).

---

## What a clean report means

A clean report means: **"No confirmed vulnerabilities were found in the areas tested,
using the methodology described."** It does NOT mean "this application is secure." Always
include:

1. What was tested (scope, domains, depth)
2. What was NOT tested (from the table above)
3. Recommended next steps (dynamic testing, domain-expert review, etc.)

---

Back to the [tree](00-map.md) · [methodology.md](methodology.md).
