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
| 9 | **Reachability of a code path under production conditions** | Can find the vulnerable *code* but may not prove it's reachable from production input without runtime tracing |

## Reduced confidence (flag but caveat)

These *can* be found but with lower confidence — report them as **"Needs Validation"**:

| # | Area | Why confidence is lower |
|---|---|---|
| 1 | **Second-order / stored injection** | Data stored now, used unsafely later in a different code path — can flag the sink but may miss the connection from source to sink |
| 2 | **Complex authorization logic** | Deeply nested RBAC/ABAC with inheritance, wildcards, and exceptions — individual gaps may not be visible from single code paths |
| 3 | **Non-obvious multi-step chains** | Can identify individual links but may miss multi-step chains that require specific ordering or preconditions |
| 4 | **Framework "magic"** | ORMs, middleware, decorators, annotations may handle security transparently — may false-positive if the framework protects it invisibly |

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

## DO NOT report as a finding

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
