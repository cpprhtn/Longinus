# Reporting & responsible disclosure

The report *is* the deliverable. A great bug poorly reported gets ignored; a clearly reported medium
gets fixed. Optimize for the reader: a developer who wants to know **what's broken, how bad, how to
reproduce, and how to fix it** — in that order.

---

## Report structure

```
1. Executive summary        ── 3–6 sentences: posture, # findings by severity, top risks, headline fix.
2. Scope & methodology      ── what was tested, how, when; what was NOT tested (stated honestly).
3. Prioritized fix list     ── the "do these first" table (Now / Soon / Eventually).
4. Findings (detailed)      ── one section per confirmed finding, ordered by severity.
5. Needs-validation         ── unconfirmed items, clearly separated.
6. Appendix                 ── tooling, raw output, references, retest results.
```

Lead with 1 and 3. Most readers act on those two and skim the rest.

## Per-finding template

```markdown
### [SEVERITY] Title that names the bug and the impact
e.g. "[Critical] IDOR in /api/orders/{id} exposes any customer's order & PII"

- **Severity:** Critical · CVSS 4.0 9.1 (vector: CVSS:4.0/AV:N/AC:L/AT:N/PR:L/UI:N/VC:H/VI:N/VA:N/...)
- **Weakness:** CWE-639 (IDOR) · OWASP API1:2023 (BOLA)
- **Location:** `src/routes/orders.ts:42` · `GET /api/orders/{id}`
- **Affected:** all authenticated users → any other user's orders

**Summary.** One paragraph: what the issue is and why it matters in this app.

**Reproduction / PoC.**
Numbered, copy-pasteable steps. Exact request(s) with a benign marker. Minimal — strip everything not
needed to trigger it. Redact real data; use your own/canary accounts.
```
1. Log in as user A (id 1001), capture the session.
2. Request `GET /api/orders/5002` (an order owned by user B).
3. Server returns user B's order incl. name, address, last-4 — no ownership check.
```

**Impact.** Concrete: what an attacker gets, at what scale. "Enumerating sequential IDs 1–N exfiltrates
the full order table (~X records, incl. PII) with a single low-priv account, no rate limiting."

**Remediation.** The fix, ideally as a patch:
```diff
- const order = await db.order.findById(req.params.id);
+ const order = await db.order.findOne({ id: req.params.id, userId: req.user.id });
+ if (!order) return res.status(404).end();   // don't leak existence
```
Plus the systemic fix (centralize authorization, add an object-ownership check in middleware) and a
regression test that fails before / passes after.

**References.** OWASP/CWE/cheat-sheet links.

**Evidence.** Screenshot/log/response excerpt (redacted).
```

## Writing principles

- **Severity-first ordering.** Critical at the top; never bury the worst bug in the middle.
- **Reproducible by a stranger.** If a dev can't follow your steps without you, the report failed.
- **Show impact, don't assert it.** "Could be serious" → demonstrate the data you could reach.
- **Fix-forward.** Every finding ends in an actionable remediation; prefer the actual diff.
- **No fluff, no fear-mongering.** Calm, precise, factual. Don't inflate severity to seem impressive.
- **Redact.** Never paste real secrets, tokens, or other users' PII into the report; mask them.
- **Reproducibility metadata.** Note dates, versions/commit hashes, and environment — bugs are
  time-sensitive.

## Responsible disclosure (third-party targets)

When the target isn't yours, you have a duty of care to users who are exposed by the bug:

1. **Use the official channel** — the program's submission form (HackerOne/Bugcrowd/Intigriti), a
   `security.txt` / `/.well-known/security.txt` contact, or a published security email. Don't DM
   random employees or post it publicly.
2. **Report privately first.** Give the vendor the details and a reasonable remediation window
   (industry norm: **90 days**, shorter if actively exploited, by mutual agreement).
3. **One bug, one report**, with everything needed to reproduce and fix.
4. **Don't over-collect.** Prove the bug with the minimum data; never exfiltrate, retain, or share
   user data. Delete any test data afterward.
5. **Coordinate before publishing.** No public writeup / PoC release until it's fixed or the agreed
   window lapses, and ideally with the vendor's nod. Request a CVE if applicable.
6. **Follow program rules** on duplicates, scope, and bounty eligibility. Out-of-scope finds → report
   as informational, don't exploit.

Never demand payment as a condition of disclosure (that reads as extortion). Bounties are a reward,
not a ransom.

## Code-audit reporting (your own repo)

For a self-audit the "report" can be lighter-weight and action-oriented:
- a prioritized checklist / issue list (one issue per finding, labeled by severity),
- the patch as a PR per fix where practical,
- a short "what I tested / what I didn't" note so you know the coverage,
- regression tests for the important ones so the bug can't silently return.

## Retest & closure

After remediation, re-run each PoC. Mark findings **Fixed / Partially fixed / Not fixed / Won't fix
(accepted risk)**, and probe for **fix bypasses** (filter that catches `../` but not `..%2f`; one
endpoint patched while its sibling isn't). Record the retest outcome in the appendix.

## References

Responsible-disclosure standards: [RFC 9116 (security.txt)](https://www.rfc-editor.org/rfc/rfc9116) ·
[ISO/IEC 29147](https://www.iso.org/standard/72311.html) (disclosure) &
[ISO/IEC 30111](https://www.iso.org/standard/69725.html) (handling) ·
[CERT/CC CVD Guide](https://vuls.cert.org/confluence/display/CVD) ·
[disclose.io](https://disclose.io/) (safe-harbor templates) ·
[HackerOne](https://docs.hackerone.com/) / [Bugcrowd](https://www.bugcrowd.com/) program guidance.
Scoring → [severity-and-triage.md](severity-and-triage.md). Full bibliography: [research/process.md](../research/process.md).

Back to the [tree](00-map.md).
