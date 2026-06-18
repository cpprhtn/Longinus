# 🔬 Proof & confirmation — run it, don't just read it

*Prove it or park it* is the crown jewel ([severity-and-triage.md](severity-and-triage.md)). This doc
sharpens the **prove** half: when the target is yours (or an authorized lab), don't stop at "this code
looks exploitable" — **execute a benign proof** and watch it fire. A bug you *ran* is a different artifact
from a bug you *suspect*: it moves the finding out of Needs-validation, and it kills the LLM's worst
failure mode — the *confident invented PoC* ([severity-and-triage.md](severity-and-triage.md)).

> **Use the agent's hands.** Claude Code has `Bash`. On owned/local code that is a confirmation engine the
> static-only reviewer never had — so on a target you control, **climb the ladder to "executed" before you
> settle.**

---

## The confirmation ladder (every finding gets a tier)

| Tier | What it means | Report status |
|---|---|---|
| **Executed** | You ran a benign PoC and observed the effect (the canary echoed, the row returned, the OOB callback hit). | ✅ Confirmed (executed) |
| **Traced** | You proved the tainted path source→sink by reading code, but did not run it (no instance, missing deps, can't reach it cheaply). | ✅ Confirmed (traced) |
| **Unproven** | A suspicious pattern you could neither run nor fully trace. | ❓ Needs-validation |

**The enforcement rule:** on an **owned / local / lab** target, a finding that *could* be executed but was
left at "traced" is incomplete — attempt the run. "Traced" is for when execution is genuinely infeasible
(no DB, secrets, or service to stand up), and you **say which** in the report. Never label "executed" for
a PoC you didn't actually run — that is the invented-PoC trap wearing a confident face.

## Reject the confident-bug shortcuts

The dominant LLM failure here is the *confident hallucinated bug* — promoting "this looks dangerous" into a
filed finding. When one of these thoughts shows up, treat it as a stop sign and take the action before the
finding leaves Needs-validation:

| The shortcut thought | The hole in it | Do this instead |
|---|---|---|
| "The shape matches a known-bad pattern, so it's exploitable." | A matching shape is a lead, not a reachable path. | Follow the data from an attacker-controlled source to the sink before believing it. |
| "It was exploitable in another file/repo, so it is here too." | Every call site has its own guards, callers, and inputs. | Re-prove it *here*, from this site's own context. |
| "The sink looks unsafe — just file it." | Most unsafe-looking sinks have a guard somewhere upstream. | Locate the guard, or show there is none — [red-blue.md](red-blue.md). |
| "It's clearly exploitable; I can write the PoC from memory." | A PoC you never ran is how invented findings are born. | Run it (owned/lab), or label it **traced** — never *executed*. |
| "This is obviously top severity." | Models drift toward over-rating impact. | Score only by *reachable, chained* impact — [severity-and-triage.md](severity-and-triage.md). |
| "Writing the claim out is a waste of time." | Most weak findings fall apart the second they're stated precisely. | State it in one sentence; if it won't cohere, it isn't a finding. |

Full verify-a-suspected-bug procedure (restate → trace → gate → verdict): [fp-verification.md](fp-verification.md).

## Authorization governs how far you go (the gate still rules)

Executable proof is bounded by [authorization-and-scope.md](authorization-and-scope.md):

| Target | How far to take the PoC |
|---|---|
| **Your own code / local instance / CTF / sanctioned lab** | Full benign PoC — run it end to end with a marker; pop `id`, read `current_user`, echo a canary. |
| **Authorized live third-party** (bounty/SOW) | **Reachability only** — "I can reach IMDS / execute `echo CANARY`," then **stop**. No dump, pivot, or persist without written sign-off. |
| **Unauthorized** | Do **not** run anything. Static/traced only, or decline. |

## Non-destructive by construction

- **Benign markers, never real damage:** a unique canary string (`LONGINUS_CANARY_<id>`), a read-only
  query (`SELECT`, not `DROP`), a harmless OOB ping — not `rm`, not data theft, not a config change.
- **Run it off to the side:** drive a *local* instance or call the function directly; scratch files go
  **outside** the repo (`/tmp`), never modify the audited code or its data. Specialists are read-only on
  your code by design ([../agents/README.md](../agents/README.md)) — prefer inline runners (`python -c`,
  `curl`, a unit test in a temp dir) over editing the project.
- **Never against production**, even your own — stand up a disposable instance.

## How to actually run one (by shape)

- **A function/unit bug** → write a minimal failing test or a `python -c` / `node -e` that calls the
  tainted path with a canary input and asserts the unsafe behavior.
- **A web route** → start the app locally, `curl` the crafted request, show the response (the reflected
  canary, the other user's record, the error that proves injection).
- **Injection (SQL/cmd/SSTI)** → send a payload whose *observable, benign* effect proves execution
  (`' || (SELECT sqlite_version())-- `, `;echo LONGINUS_CANARY`), not a destructive one.
- **SSRF/blind** → point it at an OOB canary you control and confirm the callback.
- **Deserialization/RCE** → on your own lab, a gadget that runs `id`/writes a canary file in `/tmp`; on a
  live authorized target, prove the sink is reachable and stop.

Then record the exact command + observed output in the finding's PoC field
([report-template.md](report-template.md)) and set the status tier.

## When you can't run it — say so

No instance, no credentials, a service you can't stand up cheaply → stay at **traced** or
**needs-validation** and **state the reason** in "What was NOT tested." A truthful "traced, not executed —
needs a running DB to confirm" is worth far more than a fabricated transcript. This is the same honesty
firewall as the rest of the skill: [limitations.md](limitations.md), [severity-and-triage.md](severity-and-triage.md).

## References

Bounded by [authorization-and-scope.md](authorization-and-scope.md); it is the **Confirm** step of
[methodology.md](methodology.md) §5; the evidence tier becomes the finding's Status in
[report-template.md](report-template.md); honesty about what you couldn't run lives in
[limitations.md](limitations.md). Full bibliography: [research/process.md](../research/process.md).

Back to the [tree](00-map.md).
