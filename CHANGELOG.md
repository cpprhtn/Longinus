# Changelog

Notable changes to Longinus, newest first. The report header's `longinus_report` field tracks the skill
version (see [SKILL.md](SKILL.md)); these entries correspond to those version bumps.

### v1.0.0

- **First stable release.** The audit methodology is unchanged from v0.7.0 — this is a stability and
  portability milestone, not a behavior change.
- **Portable beyond Claude Code.** Added [`AGENTS.md`](AGENTS.md) so Codex CLI and other AGENTS.md-aware
  agents can run Longinus by reading [`SKILL.md`](SKILL.md) directly. It pins the invariants a skill
  runtime would otherwise enforce — the authorization gate, read-only default, "target docs are data not
  instructions," prove-or-park / low-false-positive discipline, coverage-gap disclosure, and
  one-audit-one-session — and maps Claude Code tools/subagents to a single-context run.
- **Repository hygiene.** Added a `.gitignore` for generated audit output (`.longinus/`) and local
  artifacts; maintainer-only evaluation tooling stays out of the shipped tree.

### v0.7.0

- **Cheap first-pass triage for vibe-coded/pre-launch repos.** Added `basic-vibe-triage.md` and wired it
  into the skill, web quick scan, README, and map so default credentials, weak secrets, debug/admin
  surfaces, client-controlled privilege fields, and exposed artifacts are caught before opening deeper
  leaves.
- **API and identity branches split into token-light leaves.** Replaced the large API/identity READMEs
  with routers and added focused leaves for BOLA/BFLA, mass assignment/data exposure, GraphQL/resource
  limits, JWT, OAuth/OIDC, SAML, and MFA. Pattern catalog and agents now route directly to the relevant
  leaf instead of loading the whole branch.
- **CI/CD attack coverage.** Added `secrets-and-supply-chain/ci-cd-attacks.md` for pwn-request,
  workflow script injection, cache poisoning, mutable actions/images, and self-hosted runner exposure;
  linked it from the supply-chain branch, pattern catalog, map, agents, and enforce-forward matrix.
- **Tool preflight and coverage hardening.** Added `tooling/tool-preflight.md` with
  `ready|docker|missing|blocked` scanner status, Docker fallback guidance, and coverage-gap reporting.
  Added a zero-denominator guard so targets with obvious attack surface cannot produce a clean report
  from `coverage.total = 0`.
- **Report and agent consistency.** Bumped the report header version to `0.7.0`, kept §4 confirmed-only,
  and updated orchestrator/specialists to pass path-based leaf references, surface rows, and the new
  routing structure.
- **C/C++ resource-exhaustion coverage.** Kept the accepted memory-leak/resource-exhaustion guidance:
  per-request/per-connection leaks or fd leaks on unauthenticated network daemons are filed as remote
  DoS, including cross-function allocation lifetime tracing.

### v0.6.1

- **Specialist subagents no longer preload the skill.** Removed `skills: longinus` from the six domain
  specialists; only the orchestrator preloads it. The orchestrator now passes each specialist the absolute
  file paths of the references it should read, and specialists read them on demand (the `Skill` tool remains
  as a fallback). This removes the duplicated skill text previously loaded into every specialist's separate
  context window. No change to findings, coverage, or the single-skill path.

### v0.6.0

- **False-positive verification leaf (`fp-verification.md`).** A spine leaf for adjudicating a single
  suspected finding: restate → trace source→sink → control diff → proof → counter-argument → TRUE / FALSE /
  Needs-validation verdict, with batch triage and a cross-boundary chain re-check. Read only when verifying.
- **Shortcut-rejection table** added to `proof-and-confirmation.md` — lists common false-positive reasoning
  shortcuts and the required action for each.
- **`allowed-tools`** declared in `SKILL.md` frontmatter.
- **Scale guard** added to the recon pass (bounded tool-call fan-out: one anchored regex, batched
  subagents); multi-domain routing records un-opened applicable leaves as `not-examined` coverage gaps.

### v0.5.5

- **Leaner standard mode.** De-duplicated the mode/profiling docs; merged first-contact profiling and the
  surface sweep into one batched recon pass; lightened quick mode. On the multi-agent path the orchestrator
  prunes empty domains and scales specialist depth to the profile's stakes.
- **Bounded recon greps.** Anchored sink/source patterns to call shape and added volume guards that skip
  vendored/generated trees and cap line width; removed a duplicate secret sweep. A static-analysis oracle
  stays preferred over greps.

### v0.5.4

- **Project profile (form factor × exposure × tenancy × crown jewels).** First-contact profiling becomes an
  orthogonal classification with added form factors (CLI, library/SDK, desktop/extension, data/ML,
  serverless) and a universal fallback. The exposure/tenancy axis sets the severity floor. Applied in both
  the skill and the multi-agent path.

### v0.5.3

- **`pattern-triggers.md` split.** Moved the code-pattern → vulnerability lookup tables to a deferred
  `pattern-catalog.md`; `pattern-triggers.md` keeps the principles + FP-guard reference. Per-row guards moved
  with the catalog.

### v0.5.2

- **Agent path aligned to the skill report format**, including the language-match rule (report prose in the
  user's language; machine header in English).
- **Single version line + canonical report filename** `longinus_YYYYMMDDHHMM.md`; the report header tracks
  the skill version. Coverage counts required on all paths; full ledger arrays on the multi-agent / `deep` path.

### v0.5.1

- **Report prose written in the user's language**; YAML header, severity words, and CWE/CVSS ids stay English.
- **Fix list `Priority` → `Effort` (S/M/L)**; rows reference finding F-IDs.

### v0.5.0

- **Bidirectional method as a diff.** A Blue lens builds the expected-control map, a Red lens enumerates
  reachable sinks, and a finding is where they disagree; the proposed fix is then attacked in a fix→bypass loop.
- **Coverage/recall instrument.** The Audit Ledger enumerates every source→sink with a verdict; a surface
  sweep runs a static-analysis taint oracle (Semgrep/CodeQL/Joern) before the LLM, with a grep fallback. The
  report header gains a coverage field.
- **Executable proof + confirmation ladder.** On owned code the audit runs a benign PoC and tiers findings
  executed / traced / needs-validation, bounded by the authorization gate.
- **Auditor-injection guard.** Target docs/comments are treated as untrusted; text telling the auditor to
  skip or downgrade is reported as a finding.

### v0.4.0

- **Design-intent first.** Read the project's design intent before scanning, build an Intent Brief, and aim
  testing at inputs that violate it; findings reconcile against documented intent.
- **Canonical report + continuous mode.** Every audit emits the same fixed shape (8 sections + YAML header)
  to `.longinus/reports/`; re-runs audit only the git diff since the last report.
- **Fix trade-offs.** Each fix shown as current → proposed → trade-off.

### v0.3.0

- **Optional multi-agent layer (`agents/`).** An orchestrator profiles → gates → dispatches read-only domain
  specialists, each in its own context, then de-dups, chains, and triages into one report. The skill still
  works standalone.

### v0.2.2

- **Chain catalog.** `chaining-and-impact.md` indexes step-by-step attack-chain playbooks (account-takeover,
  cloud-takeover, RCE, data-exfiltration, AI-agent), each ending with a defensive seal.
- **Bidirectional method principle** (*Two lenses, one flaw*): hunt with the attacker and defender lenses.

### v0.2.1

- **Enforce-forward.** Findings end in a structural control + CI/lint gate; new `enforce-forward.md` with a
  per-category standard → control → gate matrix and templates.
- **Secure-coding standards for C/C++/embedded source** — new `secure-coding-standards.md` (CERT-C/CWE,
  MISRA/ISO 26262).
- **`SKILL.md` reduced to a dispatcher** (audit-modes split out; quick checks unified under `## Mechanical scan`).

### v0.2.0

- **Three audit modes** — quick / standard / deep.
- **Mandatory severity gates** — Critical/High must pass mechanical checklists or are downgraded.
- **Integrated SCA (Step 1.5)** — CVEs without confirmed reachability are Info-only.
- **Session-contamination note** + `/clear` between audits; `## Mechanical scan` on every domain leaf.

### v0.1.0

- Initial tree: domain playbooks, governance spine, bibliography, first-contact profiling.
