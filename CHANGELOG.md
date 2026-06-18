# Changelog

Notable changes to **Longinus**, newest first. The report header's `longinus_report` field tracks the
Longinus skill version (see [SKILL.md](SKILL.md)); these entries correspond to those version bumps.

### v0.5.5

- **Leaner standard mode.** De-duplicated the mode/profiling docs and merged first-contact profiling with
  the surface sweep into one batched recon pass (fewer tool round-trips); lightened quick mode. On the
  multi-agent path the orchestrator prunes empty domains and scales specialist depth to the profile's
  stakes. Net single-skill `standard` ~neutral vs v0.5.3; coverage + FP discipline unchanged.
- **Sharper, bounded recon greps.** Anchored the sink/source patterns to call-shape (less noise) and put
  them behind volume guards that skip vendored/generated trees and cap line width — so large or messy
  codebases no longer blow up context; dropped a duplicate secret sweep. No recall loss; a static-analysis
  oracle (Semgrep/CodeQL) stays preferred over greps.

### v0.5.4

- **Systematic project profile (form factor × exposure × tenancy × crown jewels).** First-contact
  profiling becomes an orthogonal classification with new form factors (CLI, library/SDK, desktop/extension,
  data/ML, serverless) and a universal fallback, so any target is classified — never a dead end. The
  exposure/tenancy axis sets the severity floor (multi-tenant → cross-tenant/BOLA is top-priority). Wired
  through both the skill and the multi-agent path.

### v0.5.3

- **`pattern-triggers.md` split for token economy.** The bulky code-pattern → vulnerability lookup tables
  moved to a deferred `pattern-catalog.md`; `pattern-triggers.md` stays a lean principles + FP-guard
  reference, so an audit reads only what it needs. FP discipline unchanged — the per-row guards travel with
  the catalog.

### v0.5.2

- **Agent path synced to the skill report format.** The multi-agent path now matches the skill's output,
  including the language-match rule (Korean request → Korean findings; the machine header stays English).
- **One version line + canonical report.** Every path writes `longinus_YYYYMMDDHHMM.md` and the report
  header tracks the skill version; coverage counts are required everywhere, with the full ledger arrays only
  on the multi-agent / `deep` path.

### v0.5.1

- **Report speaks the reader's language.** Prose is written in the language the user asked in, while the
  YAML header, severity words, and CWE/CVSS ids stay canonical English so reports still aggregate.
- **Fix list: `Priority` → `Effort`.** Drops the prescriptive Now/Soon timeline for an Effort (S/M/L)
  annotation; rows reference the finding F-IDs.

### v0.5.0

- **Bidirectional method, made real.** *Two lenses, one flaw* becomes an actual diff: a Blue lens builds
  the expected-control map, a Red lens enumerates reachable sinks, and a finding is where they disagree —
  then the proposed fix is attacked in a fix→bypass loop.
- **Coverage / recall instrument.** The new Audit Ledger enumerates every source→sink with a verdict so the
  report measures what was *not* looked at; a surface sweep puts a static-analysis taint oracle
  (Semgrep/CodeQL/Joern) in front of the LLM, with a grep fallback. The report header gains a coverage field.
- **Executable proof + confirmation ladder.** On owned code the audit runs a benign PoC and tiers each
  finding executed / traced / needs-validation, bounded by the authorization gate — never claiming
  "executed" for a PoC it didn't run.
- **Auditor injection guard.** The target's own docs/comments are treated as untrusted — text telling the
  auditor to skip or downgrade is prompt injection and becomes a finding, not an instruction.

### v0.4.0

- **Design-intent first.** Read the project's design intent before scanning, build an Intent Brief, and aim
  testing at the inputs that violate it; findings reconcile against documented intent.
- **One canonical report + continuous mode.** Every audit emits the same fixed shape (8 sections + YAML
  header) to `.longinus/reports/`, and re-runs audit only the git diff since the last report.
- **Fix trade-offs.** Every fix is shown as current → proposed → trade-off.

### v0.3.0

- **Optional multi-agent layer (`agents/`).** Run an audit as a coordinated team — an orchestrator profiles
  → gates → dispatches read-only domain specialists, each in its own context, then de-dups, chains, and
  triages into one report. Fixes session contamination structurally; the skill still works standalone.

### v0.2.2

- **Chain catalog.** `chaining-and-impact.md` becomes a hub indexing step-by-step real-world attack-chain
  playbooks (account-takeover, cloud-takeover, RCE, data-exfil, AI-agent), each ending with a defensive seal.
- **Bidirectional method.** New principle *Two lenses, one flaw* — hunt with the attacker lens and the
  defender lens; offense drives, defense seals.

### v0.2.1

- **Enforce-forward.** Findings now end in a structural control + CI/lint gate, not just a patch — new
  `enforce-forward.md` with a per-category standard → control → gate matrix and runnable templates.
- **Secure-coding standards for C/C++/embedded source** — new `secure-coding-standards.md` (CERT-C/CWE,
  MISRA/ISO 26262).
- **SKILL.md slimmed** to a lean dispatcher (audit-modes split; quick checks unified under `## Mechanical scan`).

### v0.2.0

- **Three audit modes** — quick (mechanical greps, small on-device models) / standard / deep.
- **Mandatory severity gates** — Critical/High must pass mechanical checklists or get downgraded,
  suppressing LLM severity inflation.
- **Integrated SCA (Step 1.5)** — CVEs without confirmed reachability are Info-only.
- Session-contamination warnings + `/clear` between audits; `## Mechanical scan` sections on every domain leaf.

### v0.1.0

- Initial tree: domain playbooks, governance spine, bibliography, first-contact profiling.
