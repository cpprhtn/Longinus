# Severity, triage & prioritization

A finding's value is `impact × confidence × actionability`. This doc turns a pile of observations into
a ranked, de-duplicated, trustworthy list. The goal: the developer fixes the right things first and
trusts that what you reported is real.

---

## Score with CVSS 4.0

Use [CVSS 4.0](https://www.first.org/cvss/) as the common language, but treat the number as a starting
point, not gospel — always sanity-check against real business impact.

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
unauthenticated."

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
| **Invented PoC** | a plausible-looking exploit that was never run | actually execute it; if you can't (no instance, no auth), label it unproven — don't imply you did |
| **Version-only dependency CVE** | "lib X\@1.2 has CVE-Y" with no call-path check | confirm the vulnerable function is **reached**, else mark Info/"patch anyway" |
| **Confidence laundering** | hedged uncertainty written in assertive prose | state confidence explicitly; keep the two buckets visibly separate |

When unsure, **park it** — a precise "needs validation" note is more valuable than an inflated finding.
This is the crown-jewel discipline that makes the output trustworthy.

## Prioritize the fix list

Order by `severity` then `effort` (quick critical wins first), and flag:
- **Now / blocker** — exploitable, high impact, reachable. Don't ship until fixed.
- **Soon** — real but lower impact or harder to reach.
- **Eventually / hardening** — defense-in-depth, missing headers, informational.

A founder will act on a 5-item "fix these before launch" list and bounce off a 200-row CSV. Curate.

## Triage card

```
For each candidate finding:
  reproducible?  ── no ──► "needs validation" bucket (label it, don't inflate)
       │ yes
  what's the real blast radius / asset?  ──► set business severity (may override CVSS)
       │
  same root cause as an existing finding?  ── yes ──► merge as another instance
       │ no
  assign: CWE + OWASP mapping + CVSS 4.0 vector + fix-priority (Now/Soon/Eventually)
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

These are defaults — the business-impact lens can move any of them.

## References

[CVSS 4.0](https://www.first.org/cvss/) ([calculator](https://www.first.org/cvss/calculator/4.0)) ·
[EPSS](https://www.first.org/epss/) · [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) ·
[SSVC](https://www.cisa.gov/ssvc) · [CWE](https://cwe.mitre.org/) /
[CWE Top 25](https://cwe.mitre.org/top25/) · [CAPEC](https://capec.mitre.org/) ·
[NVD](https://nvd.nist.gov/) / [CVE](https://www.cve.org/). Full bibliography: [research/process.md](../research/process.md).

Back to the [tree](00-map.md) · then write it up in [reporting-and-disclosure.md](reporting-and-disclosure.md).
