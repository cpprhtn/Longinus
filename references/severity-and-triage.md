# Severity, triage & prioritization

A finding's value is `impact × confidence × actionability`. This doc turns a pile of observations into
a ranked, de-duplicated, trustworthy list. The goal: the developer fixes the right things first and
trusts that what you reported is real.

---

## Severity anchor: the mandatory gate + the business-impact band

**Anchor every severity on the *Mandatory severity gates* (below) plus the business-impact band — those
are the reproducible parts.** An LLM builds [CVSS 4.0](https://www.first.org/cvss/)
vector strings *inconsistently* (the base-metric values drift run to run), so CVSS is an **optional
annotation, not the anchor**: if you emit a vector, do not let its exact number set the severity — the
gate and the band do, and **never block a report on producing a CVSS score**. The rough band is enough:
**Low · Medium · High · Critical** (CVSS base ≈ 0.1–3.9 · 4.0–6.9 · 7.0–8.9 · 9.0–10.0).

### CVSS 4.0 — optional common-language annotation

When you do want a number as shared vocabulary, treat it as a starting point, not gospel — always
sanity-check against real business impact.

**Base metrics (the core):**
- *Attack Vector* (Network/Adjacent/Local/Physical) — how remote is it?
- *Attack Complexity* + *Attack Requirements* — how hard / how many preconditions?
- *Privileges Required* (None/Low/High) — who can do it?
- *User Interaction* (None/Passive/Active) — does a victim have to do something?
- *Vulnerable & Subsequent System impact* on *Confidentiality / Integrity / Availability*.

**Then adjust:**
- *Threat* metrics — is it actively exploited / is exploit code public? (cross-check
  [EPSS](https://www.first.org/epss/) for probability of exploitation in the wild).
- *Environmental* metrics — what does it mean *for this target*? An IDOR exposing payment data on a
  fintech outranks the same IDOR on a throwaway demo.

Rough severity bands (CVSS base): **0.1–3.9 Low · 4.0–6.9 Medium · 7.0–8.9 High · 9.0–10.0 Critical.**

## Business-impact lens (override the math when needed)

CVSS under- or over-states real risk constantly. Re-rank by asking:
- **What does it actually expose?** (auth bypass / RCE / full-data read = top; reflected XSS needing a
  click = lower; missing security header with no exploit = informational.)
- **Crown jewels?** Anything touching auth, payments, PII, secrets, or admin gets a bump.
- **Pre-conditions realistic?** "Critical, but only with an admin session and a MitM" is not critical.
- **Blast radius?** One user vs all users vs all tenants.

State both: "CVSS 8.1 (High); rated **Critical** here because it exposes all customers' PII
unauthenticated." The mandatory gates below apply **equal downward force** — override upward only
with explicit justification.

## Mandatory severity gates (checklist before Critical/High)

An LLM auditor's dominant failure mode is **severity inflation** — defaulting to higher severity
when uncertain. Use these mechanical gates as a **mandatory final check** before assigning
Critical or High. If any required condition is not met, **force the severity DOWN one level**.

### Critical gate — ALL must be true, or downgrade to High

- [ ] **Unauthenticated or low-privilege attacker** can trigger it (PR:None or PR:Low)
- [ ] **No user interaction required** to exploit (UI:None)
- [ ] **Confirmed impact** on confidentiality, integrity, OR availability of the
      *subsequent system* or *crown-jewel data* (not just the vulnerable component)
- [ ] **Reproducible PoC exists** (not theoretical, not "if conditions X and Y hold")
- [ ] **Blast radius is multi-user or multi-tenant** (not self-only)
- [ ] **No unrealistic preconditions** (e.g., requires MitM + admin session + specific
      race timing = not Critical)

If ANY box is unchecked: **downgrade to High.** State which condition failed.

### High gate — ALL must be true, or downgrade to Medium

- [ ] **Attack complexity is realistic** (not a multi-step chain with unproven links)
- [ ] **The vulnerable code path is reachable** from production input (not dead code,
      not behind a feature flag, not test-only)
- [ ] **Impact is demonstrated, not assumed** (you showed the data/action, not "could
      potentially lead to...")
- [ ] **Not a version-only CVE** without confirmed reachability of the vulnerable function

If ANY box is unchecked: **downgrade to Medium.** State which condition failed.

### Downward override principle

The business-impact lens (above) can move severity in EITHER direction. When overriding
upward, state the business justification explicitly. When the gate fails, the downward
force is **MANDATORY** — the gate is not optional, even when "it feels Critical."

## Classify every finding

- **CWE** — the weakness type (e.g. CWE-89 SQLi, CWE-639 IDOR, CWE-918 SSRF). Enables pattern fixes.
- **CAPEC** — the attack pattern, when useful.
- **OWASP mapping** — Top 10:2025 / API Top 10:2023 / LLM Top 10:2025 / MASVS, so the dev can find the
  cheat sheet.

## De-duplicate & merge

- **Same root cause → one finding** with N instances listed. Ten reflected-XSS sinks from one missing
  encoder is *one* finding ("output encoding missing in template helper X"), not ten.
- **Distinct root cause → distinct finding**, even if symptoms look alike.
- Prefer reporting the **root cause** over each symptom — it's the fix that matters.

## Precision and recall are orthogonal (don't let one bend the other)

This firewall is the **precision** axis (*are you sure of what you filed?*). Its twin is **recall**
(*did you even look?*), instrumented by the [Audit Ledger](audit-ledger.md)'s `surface[]` coverage. They
are independent: enumerating every sink (recall) **does not** lower the bar for confirming one
(precision). A `surface` row marked `not-examined` or `reachable: unknown` is an honest **coverage gap to
disclose** (§7 of the report) — never a reason to inflate a guess into a finding. Raise coverage by
*looking harder*; keep precision by *proving harder* — neither may silently trade the other away
([audit-ledger.md](audit-ledger.md)).

## Confirmed vs unconfirmed (the trust firewall)

Maintain two buckets and never blur them:
- **Confirmed** — reproducible PoC, stated impact. These are *findings*.
- **Needs validation** — plausible but unproven (a suspicious code path you couldn't reach, a scanner
  hit you couldn't reproduce). Report as a separate, clearly-labeled section so the reader knows the
  difference. Driving false positives down is the whole game.

### LLM-auditor failure modes (self-check before you assert)

An LLM's characteristic failure is the **confident hallucinated vulnerability** — and a security tool
that cries wolf gets muted, which is worse than missing a bug. Before any finding leaves the "needs
validation" bucket, rule out these specific traps:

| Trap | What it looks like | The discipline |
|---|---|---|
| **Hallucinated bug/CVE** | asserting a vuln (or a CVE id/detail) from a pattern or memory | cite the **exact file:line** and the concrete behavior; never a remembered "this is usually vulnerable" |
| **Sink without a source** | flagging `exec`/`query`/`innerHTML` without showing attacker input reaches it | trace the **source→sink path**; no reachable taint ⇒ "needs validation," not a finding |
| **Guarded / dead / unreachable** | a real bug behind an auth check, feature flag, or dead code | prove reachability for the intended attacker role, or downgrade |
| **Invented PoC** | a plausible-looking exploit that was never run | actually execute it on owned/local code ([proof-and-confirmation.md](proof-and-confirmation.md)); if you can't (no instance, no auth), label it `Confirmed (traced)` or unproven — don't imply you did |
| **Version-only dependency CVE** | "lib X\@1.2 has CVE-Y" with no call-path check | confirm the vulnerable function is **reached**, else mark Info/"patch anyway" |
| **Confidence laundering** | hedged uncertainty written in assertive prose | state confidence explicitly; keep the two buckets visibly separate |

When unsure, **park it** — a precise "needs validation" note is more valuable than an inflated finding.
This is the crown-jewel discipline that makes the output trustworthy.

> **Canonical source.** This table is the single source of truth for false-positive / "DO NOT report"
> discipline. The local restatements in [pattern-triggers.md](pattern-triggers.md) (audit-time guards),
> [limitations.md](limitations.md) (what static analysis can't reach), and [SKILL.md](../SKILL.md)
> (operating principles) intentionally repeat it for weaker models — **keep them in sync with this
> table; if they ever disagree, this one wins.**

## Design-intent reconciliation (is it deliberate?)

Before finalizing a mid/low finding, check it against the project's **Intent Brief**
([design-intent.md](design-intent.md)):
- **Matches a *documented* accepted-risk** → downgrade to **Informational**, *cite the doc/comment*.
- **Contradicts** stated intent → keep or **raise** severity (the system does what it was *not* designed to).
- Relies on an **undocumented** assumption → it **stays a finding** ("confirm this is intended").

> Intent gives context, never absolution: a mid-risk issue is "deliberate" only if it's *written down*.

## Fix trade-offs are informational

Each fix is shown as **current → proposed → trade-off** — the perf/UX cost of applying the proposed
change ([reporting-and-disclosure.md](reporting-and-disclosure.md)). The trade-off **never lowers
severity**: Critical/High get fixed regardless; it only informs *how* to fix, not *whether*.

## Exposure & tenancy set the severity floor

Severity is impact × reachability — and *reachability* is set by the project's **exposure/tenancy** profile
([audit-modes.md](audit-modes.md) Step 2), so apply it before scoring:
- **Public-internet** → remote-reachable bugs keep full severity; **internal-only / local** → drop most
  remote attacks toward Low (but local-privilege, supply-chain, and malicious-insider paths rise).
- **Multi-tenant** → **cross-tenant / BOLA / isolation breaks are top-priority and rated up** (one tenant
  reading another's data leans Critical); **single-tenant** → those collapse.
- **Crown jewels present** (auth / money / PII / secrets) → a reachable bug touching them takes the higher
  severity band; **nothing at stake** → cap most findings lower.

This is *context, not invention*: it tunes the reachability/impact inputs the gates already use — it never
manufactures a finding, and never lowers a confirmed sink below its proven impact.

## Prioritize the fix list (two axes — and stay in your lane)

Order the fix list by **severity** (how bad), and annotate each with **effort** (how cheap to fix:
**S** = a line/config · **M** = one module · **L** = re-architecture). Those two let the reader *sequence*
— a one-line Critical and a trivial quick-win go before a hard High of the same-or-lower severity.

> **Rate risk, not their roadmap.** *When* to fix (the remediation SLA) is the owning team's + vuln-
> management's call, driven by policy and business context (compensating controls, risk acceptance), not
> something the auditor stamps per finding. So the report carries **severity + effort**, *not* a
> prescriptive "do this Now" — surfacing a timeline as a mandate is overstepping.

A founder acts on a curated 5-item list and bounces off a 200-row CSV. Curate; lead with the chain.

## Triage card

```
For each candidate finding:
  reproducible?  ── no ──► "needs validation" bucket (label it, don't inflate)
       │ yes
  what's the real blast radius / asset?  ──► set business severity (may override CVSS)
       │
  same root cause as an existing finding?  ── yes ──► merge as another instance
       │ no
  assign: CWE + OWASP mapping + CVSS 4.0 vector + fix-effort (S/M/L)
       │
  if Critical or High: run the mandatory severity gate checklist (above)
       │                if any condition fails ──► downgrade one level, state why
```

## Severity quick-reference (typical ratings)

| Class | Typical band | Notes |
|---|---|---|
| Unauth RCE / SQLi / auth bypass | Critical | top of the list, always |
| SSRF to cloud metadata → creds | Critical | via chain |
| IDOR/BOLA on sensitive data | High–Critical | scale by data + scope |
| Stored XSS (other users) | High | lower if self-only |
| Insecure deserialization (reachable) | High–Critical | RCE potential |
| Exposed secret (live, privileged) | High–Critical | rotate immediately |
| CSRF on state change | Medium–High | depends on action |
| Reflected XSS (needs interaction) | Medium | |
| Missing security headers / verbose errors | Low–Info | hardening |
| Outdated dependency, no reachable path | Low–Info | note + patch anyway |

These are defaults before applying the mandatory gates. Apply the gate checklist to every
Critical/High finding before finalizing.

## References

[CVSS 4.0](https://www.first.org/cvss/) ([calculator](https://www.first.org/cvss/calculator/4.0)) ·
[EPSS](https://www.first.org/epss/) · [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) ·
[SSVC](https://www.cisa.gov/ssvc) · [CWE](https://cwe.mitre.org/) /
[CWE Top 25](https://cwe.mitre.org/top25/) · [CAPEC](https://capec.mitre.org/) ·
[NVD](https://nvd.nist.gov/) / [CVE](https://www.cve.org/). Full bibliography: [research/process.md](../research/process.md).

Back to the [tree](00-map.md) · then write it up in [reporting-and-disclosure.md](reporting-and-disclosure.md).
