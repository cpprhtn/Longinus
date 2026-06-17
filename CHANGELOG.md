# Changelog

Notable changes to **Longinus**, newest first. The report header's `longinus_report` field tracks the
Longinus skill version (see [SKILL.md](SKILL.md)); these entries correspond to those version bumps.

### v0.5.2

- **Agent path synced to the skill report format** — the multi-agent orchestrator drops a stale
  "prioritized" wording, and the v0.5.1 language-match rule now propagates to the Red specialists, so a
  Korean audit returns Korean finding prose from the **multi-agent** path too (Blue stays English — it
  builds the machine-layer control map).
- **One version line + canonical filename** — every path writes `longinus_YYYYMMDDHHMM.md` (UTC), and the
  report header's `longinus_report` field carries the **Longinus skill version** (now `0.5.2`) instead of
  an independent document-schema number (was `1.2`) — skill, agents, and report share one version.
- **Coverage baseline vs. ledger arrays** — the `coverage:` counts are required in every report; the full
  `surface[]`/`controls[]` ledger arrays ride in the header only on the multi-agent / `deep` path, while
  the single-skill/`standard` baseline accounts for each sink in §7 prose. Resolves a skill-vs-agent report
  divergence.

### v0.5.1

- **Report speaks the reader's language** — the audit writes its prose (summary, findings, fixes,
  headings) in the **language the user asked in** (a Korean request → a Korean report), while the YAML
  header, severity words, and CWE/CVSS ids stay canonical English so reports still aggregate across a
  fleet (`references/report-template.md` rule 8).
- **Fix list: `Priority` → `Effort`** — the §3 fix list drops the prescriptive "do this Now/Soon" column
  (the remediation timeline is the team's call, not the auditor's) and adds an **Effort** (S/M/L)
  annotation instead, so you can sequence quick wins first. Rows now reference the finding F-IDs.

### v0.5.0

- **Bidirectional method, made real** — *two lenses, one flaw* is now an actual **diff**, not a footnote.
  A **Blue** lens builds the expected-control map from the design intent; a **Red** lens enumerates
  reachable sinks; a finding is a reachable sink whose expected control is missing or bypassed — then the
  proposed fix is attacked in a **fix→bypass loop** until it holds. New spine doc
  `references/red-blue.md`; new `agents/longinus-blue.md` runs first and feeds the Red specialists.
- **Coverage / recall instrument** — the new **Audit Ledger** (`references/audit-ledger.md`) enumerates
  every source→sink with a verdict, so the report measures *what was not looked at*. Recall (did you
  look?) is now the explicit, orthogonal twin of precision (are you sure?) — neither can silently trade
  the other away. A **surface sweep** (`references/audit-modes.md`) puts a static-analysis taint oracle
  (Semgrep/CodeQL/Joern) in front of the LLM as the recall + reachability engine, with a grep fallback.
- **Executable proof + confirmation ladder** — on owned/local code the audit **runs a benign PoC** and
  tiers each finding `Confirmed (executed)` / `Confirmed (traced)` / `Needs-validation`, bounded by the
  authorization gate (`references/proof-and-confirmation.md`); never claims "executed" for a PoC it didn't
  run.
- **Auditor injection guard (eats its own dog food)** — the target repo's docs/comments/`SECURITY.md` are
  treated as **untrusted input**; text that tells the auditor to skip, downgrade, or "report nothing" is
  indirect prompt injection and becomes a finding, never an instruction, and can't launder a real bug via
  the design-intent downgrade path (`references/design-intent.md`).
- **Coverage in the report header** — the report YAML header gains a `coverage` field (examined / total sinks), and a
  finding's Status carries its evidence tier (executed / traced); the 8 fixed sections are unchanged.

### v0.4.0

- **Design-intent first** — read the project's design intent (`CLAUDE.md`/README/ADRs/`SECURITY.md`/
  comments) *before* scanning, build an **Intent Brief** (purpose · designed trust boundaries · stated
  assumptions · documented accepted-risks), and aim testing at the input vectors that **violate** it.
  Findings reconcile against intent: documented accepted-risk → Informational (cited); contradicts intent
  → real/higher; *undocumented* assumption → still a finding. (`references/design-intent.md`)
- **Fix trade-offs** — every fix is shown as **current → proposed → trade-off** (the perf/UX cost of the
  change); informational — severity still decides *whether* to fix.
- **Report as a file + continuous/diff mode** — every audit writes `.longinus/reports/longinus_YYYYMMDDHHMM.md`;
  re-runs audit only the `git diff` since the last report and append a delta (drive it from cron/CI or
  `/schedule`). (`references/continuous-audit.md`)
- The orchestrator builds the Intent Brief once and passes it to every specialist.
- **One canonical report template** (`references/report-template.md`) — every audit emits the *same*
  fixed shape (8 sections + a machine-readable YAML header), ending report inconsistency across people
  and projects and making reports aggregatable/diffable.

### v0.3.0

- **Optional multi-agent layer** (`agents/`) — run a Longinus audit as a coordinated team. A
  **longinus-orchestrator** profiles → gates authorization → dispatches 5 read-only domain specialists
  (`secrets`, `web`, `api-identity`, `cloud`, `ai`), each in its **own context window**, then de-dups →
  chains → triages → one ranked report. This fixes session contamination *structurally* (no cross-domain
  bias) and runs specialists in parallel. Install by copying `agents/*.md` into `~/.claude/agents/`; the
  skill still works **standalone**. See [agents/README.md](agents/README.md).

### v0.2.2

- **Chain catalog** — `chaining-and-impact.md` is now a hub indexing deep, step-by-step real-world
  attack-chain playbooks under `references/chaining/` (account-takeover, cloud-takeover, rce-chains,
  data-exfiltration, ai-agent-chains). Each is **offense-driven** and ends with a 🛡️ *defensive seal* —
  the single gate that breaks the chain.
- **Bidirectional method** — new operating principle *"Two lenses, one flaw"*: hunt with the attacker
  lens (where it breaks) **and** the defender lens (what control must be there); **offense drives,
  defense seals**. `enforce-forward.md` reframed as the defender lens of that method.

### v0.2.1

- **Enforce-forward** — findings now end in a *structural control + CI/lint gate*, not just a patch.
  New `references/enforce-forward.md` with a per-category **standard → control → gate** matrix and
  runnable `references/templates/` (validation-layer, ci-gates, policy-as-code, pre-commit-and-secrets);
  the *Fix-forward* principle is upgraded to *Enforce-forward*. Each gate is framed by the attack it kills.
- **Secure-coding standards for C/C++/embedded source** — new `references/secure-coding-standards.md`
  (CERT-C/CWE for security · MISRA/ISO 26262 for safety), the source-compliance companion to the
  binary-exploitation / reverse-engineering branches.
- Per-category Enforce-forward pointers across domain READMEs; identity gains NIST 800-63 / FAPI.
- SKILL.md slimmed to a lean dispatcher via an `audit-modes.md` split; quick-check sections unified
  under `## Mechanical scan`; `severity-and-triage.md` set as the canonical false-positive source.

### v0.2.0

- Three audit modes: quick (mechanical grep scans, small on-device models 11B+) / standard / deep
- Mandatory severity gates — Critical and High findings must pass mechanical checklists or get
  downgraded, suppressing LLM severity inflation
- Integrated dependency health checks (SCA) as Step 1.5 in the audit flow; CVEs without confirmed
  reachability are Info-only
- Session contamination warnings and `/clear` recommendation between audits
- `## Mechanical scan` sections added to all 22 domain leaves for small-model compatibility
- `attacker-mindset.md` merged into `pattern-triggers.md` (token optimization)
- Domain quick-checks moved from SKILL.md to each domain README
- Routing deduplication between SKILL.md and 00-map.md

### v0.1.0

- Initial tree: domain playbooks, governance spine, bibliography, first-contact profiling
