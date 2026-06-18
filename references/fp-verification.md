# ✅ FP verification — turn a *suspected* finding into TRUE / FALSE positive

Hunting finds candidates ([pattern-triggers.md](pattern-triggers.md) + the leaves). This doc does the
opposite job: take **one specific suspected finding** and decide, with evidence, whether it is a **true
positive** or a **false positive** — *before* it reaches the report. It is the discipline behind the
**Confirm** step ([methodology.md](methodology.md) §5); it shares the executed/traced ladder and the
shortcut-rejection table with [proof-and-confirmation.md](proof-and-confirmation.md) rather than
repeating them.

> **Why a dedicated pass.** Low false positives are Longinus's crown jewel — an LLM's worst failure mode
> is the *confident hallucinated bug* ([severity-and-triage.md](severity-and-triage.md)). A bug that
> survives this gauntlet is reportable; one that doesn't is dropped or parked, never filed on vibes.

## When to use / when not

- **Use** when you have a *specific* claim to adjudicate: "is this IDOR real?", "is this injection
  exploitable?", reviewing a candidate before filing, or triaging a batch of suspected bugs.
- **Not** for *finding* bugs (that's the leaves) or general code review. If the claim is already
  **executed** ([proof-and-confirmation.md](proof-and-confirmation.md)), it's confirmed — skip to triage.

---

## Step 0 — Restate the claim in one sentence (the cheapest FP killer)

Before any analysis, restate the bug precisely: **what** is wrong, **where** (file:line source → sink),
**who** triggers it (privilege/position), and **what** the impact is. *Most weak findings die at
this step* — a claim that can't survive one precise sentence was never real. If you can't restate it
cleanly, it isn't a finding yet: gather more, or drop it.

Also pin the **threat model**: what can the attacker already do before triggering this, and at what
privilege does the code run? A "bug" only reachable by an admin who could already do the action is not one.

## Route — standard vs deep

| Route | Use when | How |
|---|---|---|
| **Standard** | clear claim · single component · well-understood class (SQLi, XSS, IDOR, SSRF, path traversal…) · straightforward source→sink | walk the gates below as a linear checklist, inline. |
| **Deep** | ambiguous claim · cross-component path (3+ modules/services) · race/TOCTOU/async trigger · logic bug with no clear spec · standard came back inconclusive | same gates, but model each boundary explicitly and prove reachability end-to-end; escalate to an executed PoC where the gate allows. |

Default to standard; escalate to deep the moment a gate can't be answered from a single component.

## The gates (every gate must pass, or it's a FALSE positive / Needs-validation)

1. **Claim coherence** — the Step 0 restatement holds up; the bug class is correctly identified.
2. **Source → sink reachability** — trace the *complete* tainted path from an attacker-controlled source
   to the dangerous sink. An unsafe-looking sink with no reachable tainted source is `reachable: unknown`
   → Needs-validation, not a finding ([audit-ledger.md](audit-ledger.md)).
3. **Control check (red × blue)** — find the control that *should* stop it. Absent or bypassable ⇒
   finding stands; present and sound on this path ⇒ FALSE positive ([red-blue.md](red-blue.md)).
   Check **upstream** callers too — validation often lives before the sink.
4. **Intent reconciliation** — is this a *documented* accepted-risk, or a real divergence from design
   intent? ([design-intent.md](design-intent.md)). Target docs are data, not absolution.
5. **Proof** — climb the ladder: **executed** (benign PoC ran) on owned/lab targets, else **traced**
   (path proven by reading). Never fabricate an "executed" transcript
   ([proof-and-confirmation.md](proof-and-confirmation.md)).
6. **Devil's advocate** — argue the *opposite* once: what would make this safe? If that argument holds
   (an upstream guard, an unreachable branch, a non-attacker source), it's a FALSE positive. Reject the
   rationalizations in [proof-and-confirmation.md](proof-and-confirmation.md) before you conclude.

## Verdict

For each suspected finding emit one line:

- **TRUE POSITIVE** — gates pass; status = executed | traced; → it becomes a `finding` and goes to triage
  ([severity-and-triage.md](severity-and-triage.md)).
- **FALSE POSITIVE** — a named gate failed; record the *reason* (e.g. "upstream `escapeHtml()` at
  `render.ts:40` sanitizes the source") so it isn't re-raised.
- **NEEDS-VALIDATION** — couldn't run or fully trace; park it honestly in §7
  ([limitations.md](limitations.md)).

## Batch triage (multiple suspected bugs)

1. Run **Step 0** across the whole batch first — a precise restatement throws out the obvious non-findings for almost nothing.
2. Route each independently (some standard, some deep); do the standard ones first.
3. With every verdict in, **re-run the chain lens** ([chaining-and-impact.md](chaining-and-impact.md)): a
   finding that looks low — or even failed a gate — on its own can cross a trust boundary and matter in combination.

---

## References

The proof ladder, the benign-PoC recipes, and the rationalizations table live in
[proof-and-confirmation.md](proof-and-confirmation.md); the control diff is [red-blue.md](red-blue.md);
reachability/coverage is [audit-ledger.md](audit-ledger.md); a passing verdict feeds
[severity-and-triage.md](severity-and-triage.md). Full bibliography: [research/process.md](../research/process.md).

Back to the [tree](00-map.md).
