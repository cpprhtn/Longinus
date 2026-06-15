# Engagement methodology (the lifecycle)

A repeatable loop that works for a code audit, a bounty target, or a CTF challenge. It blends
**PTES**, **NIST SP 800-115**, and **OWASP WSTG/MASTG** with the bug-bounty recon→report flow. The
shape is the same; only the depth and the "active vs static" emphasis change.

```
0. Authorize  →  1. Scope & profile  →  2. Recon/discovery  →  3. Map & threat-model
        →  4. Test by domain  →  5. Confirm (PoC)  →  6. Chain  →  7. Triage  →  8. Report  →  9. Retest
```

Always loop back: a confirmed bug often reveals new surface (a leaked key → new endpoints; an IDOR →
new object ranges).

---

## 0. Authorize
Pass the gate in [authorization-and-scope.md](authorization-and-scope.md). No gate, no active testing.

## 1. Scope & profile
Characterize the target precisely (see SKILL.md Step 1): form factor, stack, trust boundaries, crown
jewels. Write down the **in-scope** and **out-of-scope** lists. For a repo, inventory languages,
frameworks, entry points, datastores, and dependencies from the manifests.

The deliverable of this phase is a one-paragraph target model plus an **input→sink→asset** sketch:
where untrusted data enters, where it lands (DB query, shell, file path, HTML, deserializer, LLM
prompt, cloud API), and what's valuable behind it.

## 2. Recon / discovery
- **Dynamic target:** enumerate the surface — see [recon/](recon/README.md). Subdomains, live hosts,
  ports/services, content/endpoints, parameters, JS-revealed routes, historical URLs, tech
  fingerprints.
- **Code target:** build the surface from the source — route tables, controllers/handlers, GraphQL
  schema, message consumers, cron jobs, webhooks, and every place user input is read. Grep for sinks
  (`exec`, `query`, `eval`, `pickle.loads`, `Runtime.exec`, `innerHTML`, `os.system`,
  template renderers, `fetch(<userurl>)`).
- **CTF:** read the prompt, identify the category, pull the provided files, and run first-pass triage
  tools (`file`, `strings`, `binwalk`, source review).

## 3. Map & threat-model
Turn discovery into a prioritized plan. For each entry point, ask: *who can reach it, with what
privilege, and what's the worst it touches?* Prioritize unauthenticated → authenticated-low →
authenticated-high, and anything touching crown jewels. Pick the [tree branches](00-map.md) that match.
Lightweight STRIDE (Spoofing/Tampering/Repudiation/Info-disclosure/DoS/Elevation) per trust boundary
is enough — don't over-model.

## 4. Test by domain
Work the selected branch playbooks. Each leaf gives find→confirm→fix steps and representative test
cases. Discipline:
- **Breadth then depth:** sweep the surface for candidate issues, then deep-dive the promising ones.
- **One variable at a time** when probing, so you understand *why* something responded.
- **Authenticated and unauthenticated** passes — and across roles (horizontal & vertical).
- **Default non-destructive:** canary values, OOB callbacks, read-only proofs (see the gate).

## 5. Confirm (prove it or park it)
A suspicion is not a finding. Confirm with a **minimal, reproducible PoC**:
- exact request/input that triggers it, the observed vs expected behavior, and a marker proving cause;
- for blind bugs, an out-of-band signal (DNS/HTTP callback to a collaborator you control);
- for code findings, the tainted path from source to sink, ideally a failing test.

Anything you can't confirm goes to a separate **"needs validation"** list — never reported as fact.
This is the discipline that makes Longinus output trustworthy instead of scanner noise.

## 6. Chain
Combine issues — real impact is usually a chain (open-redirect + OAuth → token theft; SSRF + cloud
metadata → credential theft → IAM priv-esc). Explicitly try to escalate **every** low/medium across a
trust boundary; the chained severity is what you report. This is a first-class discipline, not an
afterthought: work it from [chaining-and-impact.md](chaining-and-impact.md).

## 7. Triage
Score and de-duplicate per [severity-and-triage.md](severity-and-triage.md): CVSS 4.0 + business
impact, merge same-root-cause instances, rank ruthlessly. Separate confirmed from unconfirmed.

## 8. Report
Write it up per [reporting-and-disclosure.md](reporting-and-disclosure.md): executive summary,
prioritized fix list, then per-finding detail with PoC and remediation. For code audits, propose the
patch. For third-party targets, follow responsible disclosure.

## 9. Retest
After fixes, re-run the specific PoCs to confirm closure and check for fix-bypasses (a blocklist that
filters `../` but not `..%2f`; a patched endpoint with a sibling that's still vulnerable).

---

## Static (code) vs dynamic (running) mode

| | Static / code audit | Dynamic / live testing |
|---|---|---|
| Authorization | implicit (own code) | explicit (scope/permission) |
| Recon | inventory from source | enumerate the surface |
| Finding | tainted source→sink path | observed exploitable behavior |
| PoC | failing test / annotated code path | request/response or callback |
| Strength | full coverage, sees logic | proves real-world exploitability |
| Weakness | can't always prove reachability | only sees exposed surface |

Best results use **both**: code review finds the bug class and the exact line; dynamic testing proves
it's reachable and impactful. For vibe-coded apps, lead with static (you own it, it's fast, it's
comprehensive) and confirm the scary ones dynamically on a local instance.

## Time-boxing & coverage

Don't rabbit-hole. Allocate effort by expected value: unauthenticated surface and crown-jewel paths
first. Track coverage against the relevant checklist (WSTG for web, API Top 10 for APIs, LLM Top 10 for
AI, MASVS for mobile) so the report can state what *was* and *wasn't* tested — gaps stated honestly are
part of a professional deliverable.

## References

This lifecycle blends [PTES](http://www.pentest-standard.org/),
[NIST SP 800-115](https://csrc.nist.gov/pubs/sp/800/115/final),
[OWASP WSTG](https://owasp.org/www-project-web-security-testing-guide/) (web) and
[OWASP MASTG](https://mas.owasp.org/) (mobile) with the bug-bounty recon→report flow. Coverage
checklists per domain live in each branch README. Full bibliography: [research/process.md](../research/process.md).

Back to the [tree](00-map.md).
