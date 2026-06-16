# 🔁 Continuous & incremental audits — report files + history

Make Longinus **operational**: every audit emits a timestamped Markdown report to a default directory,
and re-runs use the prior reports to audit **only what changed**. This turns a one-shot review into a
scheduled / CI gate that watches a project over time.

## Every audit writes a report file (the output contract)

End **every** audit by writing the report to:

```
.longinus/reports/longinus_YYYYMMDDHHMM.md      # UTC timestamp
```

- **Default dir** `.longinus/reports/` (add it to `.gitignore` unless you want the history in version
  control). The path is configurable; keep it stable so re-runs can find prior reports.
- **Content** = [report-template.md](report-template.md) emitted **verbatim** — its fixed 8 sections
  plus the machine-readable **YAML header** (target · commit SHA · date · mode · scope · `overall_risk`
  · severity `counts` · `coverage`). Same shape every project and run, so reports aggregate and diff across a fleet.
  The *why* behind each field is in [reporting-and-disclosure.md](reporting-and-disclosure.md).
- **Mandatory:** an audit with no report file is *incomplete*. In headless/CI runs, the LLM may skip the
  write — so state it explicitly in the prompt (*"write the report to `.longinus/reports/`"*) to make
  the run deterministic.

## Incremental / diff mode (use the history)

On a re-run, don't re-audit the whole tree:

1. Find the **latest prior report** in `.longinus/reports/`; read the **commit SHA** it recorded.
2. `git diff <prior-sha>..HEAD --name-only` → audit **only the changed files** (and their reachable
   callers / the sinks they touch).
3. **Re-check the prior open findings** — for each: **fixed · still-open · regressed**?
4. Write a new report whose top is a **delta**: *new · fixed · still-open · regressed*, followed by the
   rolling list of open findings.

This cuts a 2-hourly job to seconds and answers the question that matters on every change: **"what did
*this* change introduce?"** Pairs with the *continuous/diff* mode in
[audit-modes.md](audit-modes.md) — a fast mechanical scan of the diff plus a re-check.

## Scheduling is external (the skill can't schedule itself)

A skill is guidance, not a daemon — drive it from outside:

- **cron / CI on push:** `claude -p "run a Longinus incremental audit; write the report to .longinus/reports/"`
  (headless) on a timer or a pre-merge hook.
- **Claude Code routines:** `/schedule` a recurring run.
- **CI gate:** fail the build when the delta contains a new **High/Critical** — the defensive back-half
  of [enforce-forward.md](enforce-forward.md). A fix's **trade-off** (current → proposed → perf/UX)
  belongs in the report so a human can weigh the gate.

## References

The report format lives in [reporting-and-disclosure.md](reporting-and-disclosure.md); modes in
[audit-modes.md](audit-modes.md); the per-run Intent Brief in [design-intent.md](design-intent.md).
Full bibliography: [research/process.md](../research/process.md). Back to [tree](00-map.md).
