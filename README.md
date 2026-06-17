# 🗡️ Longinus

> *The spear that pierces your own armor first.*  
> An offensive-security skill that plays attacker so the **defender** finds the hole before a real adversary does.

**English** · [한국어](README.ko.md)

**Longinus is a [Claude Code Skill](https://docs.claude.com/en/docs/claude-code/skills) that turns
Claude into a security auditor for your project.** It plays attacker to find vulnerabilities, then
reports **prioritized findings — each with a reproducible proof-of-concept and a concrete fix.** It's a
tree of offensive-security playbooks plus an orchestration brain ([SKILL.md](SKILL.md)) that aims them
at a target (web, APIs, identity, cloud, mobile, AI/LLM, binaries, crypto, forensics — as deep as the
target needs).

## Install

Longinus is a Claude Code Skill — Claude Code auto-discovers any skill placed in a `skills/` directory.

```bash
# Per-project (commit it with your repo, share with the team)
git clone https://github.com/cpprhtn/Longinus.git .claude/skills/longinus

# Personal (available in every project you work on)
git clone https://github.com/cpprhtn/Longinus.git ~/.claude/skills/longinus
```

Clone the **whole tree** so `SKILL.md` sits at the folder root with `references/` and `research/`
beside it — the internal links are relative, so copying individual files breaks it.

**Optional — multi-agent layer.** The skill works standalone. For a coordinated audit where each domain
runs in its **own context** (no cross-domain bias, parallel specialists), copy the bundled
[subagents](agents/README.md) where Claude Code discovers them, then restart Claude Code:

```bash
# Per-project (when the skill is cloned to .claude/skills/longinus)
mkdir -p .claude/agents && cp .claude/skills/longinus/agents/*.md .claude/agents/

# Or personal — available in every project (after a personal skill install)
cp ~/.claude/skills/longinus/agents/*.md ~/.claude/agents/
```

Then ask *"run a Longinus audit"* and the **orchestrator** dispatches the specialists. See
[agents/README.md](agents/README.md).

## Usage

After installing, invoke it **three ways**:

```text
1) Slash command   /longinus            ← standard audit
                   /longinus quick      ← fast mechanical scan
                   /longinus deep       ← full multi-domain sweep

2) Just ask        "Audit this repo before I launch."
                   "Review the security of this FastAPI API."
                   "Red-team the prompt injection surface of my agent."

3) Multi-agent     "run a Longinus audit"   (if you installed agents/ above)
                   → the orchestrator dispatches the domain specialists
```

**It's stack-agnostic** — Python/FastAPI, Node/Express, React Native, Go, Terraform/cloud, an LLM/RAG
app — because it **profiles the target first**, then routes to the right playbooks. It guides Claude to
audit your code (read-only by default); it is *not* an automatic scanner that runs on its own. Every run
writes a fixed-format report to `.longinus/reports/longinus_YYYYMMDDHHMM.md`. Before testing anything you
don't own, clear the [authorization gate](references/authorization-and-scope.md).

> ⛔ **Offense in service of defense.** Longinus is for code you own, targets you're explicitly
> authorized to test, and CTF/lab environments — never for unauthorized systems. See
> [docs/ethics.md](docs/ethics.md).

## Skill modes

Pick how deep to go — pass it on the command, or just say it. Default is **standard**.

| Mode | Invoke | What it does |
|---|---|---|
| **quick** | `/longinus quick` · *"run a quick security audit"* | Mechanical grep scans (each leaf's `## Mechanical scan`), fixed severities, no CVSS/chaining — one findings table. Built for **CI gates** and **small on-device models (11B+)** like Qwen 3 8B / Llama 3.1 8B (mechanical instructions, no open-ended reasoning). |
| **standard** | `/longinus` · *"audit this repo"* | **Default.** Profile → route → full leaf analysis → PoC → triage (CVSS 4.0) → report. |
| **deep** | `/longinus deep` · *"deep audit this repo"* | Standard + all secondary domain quick-checks + cross-domain [chaining](references/chaining-and-impact.md). |
| **continuous** | `/longinus continuous` · cron/CI | Audits only the `git diff` since the last report + re-checks prior findings → appends a delta ([continuous-audit.md](references/continuous-audit.md)). |

> Full mode procedures live in [references/audit-modes.md](references/audit-modes.md). If you run a local
> model for security checks, start with **quick**.

## What makes it different

Not just another playbook pack:

- **Reasons from principles, not a checklist.** Instead of walking a fixed list, it derives bugs from
  six root principles that *generate* vulnerability classes — trust boundaries, parser differentials,
  confused deputy, state & time, encoding, the developer's unstated assumptions →
  [pattern triggers & attacker principles](references/pattern-triggers.md). It catches classes that
  don't even have a name yet.
- **Reads your design intent first.** Builds an *Intent Brief* from your `CLAUDE.md`/README/ADRs, then
  hunts where the implementation diverges from it — and won't flag a *documented* deliberate decision as
  a bug → [design-intent](references/design-intent.md).
- **Rates findings by chained impact, not in isolation.** Issues that are individually "low" can
  compose into account takeover; that chain is treated as one **Critical** →
  [chaining](references/chaining-and-impact.md).
- **Bidirectional by design — the finding is a diff.** A *Blue* lens maps the controls that *should*
  guard each trust boundary; a *Red* lens finds the reachable sinks; a vulnerability is where they
  disagree — then the proposed fix is attacked until it can't be bypassed →
  [red × blue method](references/red-blue.md).
- **Measures coverage, not just bugs.** Every source→sink goes into an [Audit Ledger](references/audit-ledger.md)
  with a verdict, so the report states *what was not looked at* (recall) — an un-examined sink is a
  disclosed gap, never a silent "all clear."
- **Ruthlessly suppresses false positives.** No reproducible PoC, no confirmed finding. Mandatory
  severity gates force Critical/High findings *down* when preconditions aren't met — an LLM's weakest
  point is severity inflation → [severity & triage](references/severity-and-triage.md).
- **Confirms by running it, not guessing.** On code you own, it uses the agent's shell to **execute a
  benign PoC** and label the finding `Confirmed (executed)` vs `(traced)` — turning "looks exploitable"
  into proof, bounded by the authorization gate → [proof & confirmation](references/proof-and-confirmation.md).
- **Hardened against a repo that attacks the auditor.** The target's own `README`/comments/`SECURITY.md`
  are *untrusted data* — text telling the auditor to "skip this" or "report nothing" is indirect prompt
  injection, so it becomes a finding, never an instruction (it eats its own dog food) →
  [design-intent](references/design-intent.md).
- **One fixed report shape, every time.** Every audit emits the same template (machine-readable header +
  fixed sections) so reports are consistent and comparable across projects →
  [report template](references/report-template.md).
- **Honest about its limits.** A [limitations doc](references/limitations.md) says what static/LLM
  analysis *cannot* find, so a clean report never implies "no vulnerabilities exist."

## Why

- **AI-generated code ships fast and insecure.** You need a systematic way to audit it before launch.
- **Security knowledge is scattered.** Instead of hunting through OWASP, bug-bounty writeups, and CTF
  lore every time, follow one proven methodology.
- **Scanners drown you in noise.** Longinus follows a *prove-it-or-park-it* rule and reports only what
  it can actually demonstrate.

## Dependency CVE checks

Longinus does not have its own CVE database. It delegates to the ecosystem's SCA tools and applies
its own severity rules on top:

```
Step 1   Identify the stack (package.json → Node, requirements.txt → Python, …)
Step 1.5 Run the matching SCA tool:
           Node → npm audit    Python → pip-audit / osv-scanner
           Go   → govulncheck  Rust   → cargo audit
           Ruby → bundler-audit  Java → trivy fs .
                ↓
         SCA tool queries its CVE database
         (GitHub Advisory DB, OSV, NVD, etc.)
                ↓
         Longinus filters the results:
           reachable code path confirmed  → normal severity triage
           reachability NOT confirmed     → Info / "patch anyway" only
           hallucinated CVE (unverifiable)→ DO NOT report
                ↓
         Reported in a separate "Dependency Health" section
```

The SCA tools must be installed in the environment. `npm audit` comes with Node; others
(`pip-audit`, `trivy`, `govulncheck`, …) require separate installation. If a tool is missing,
that check is skipped — this is an environment constraint, not a skill limitation.

## Documentation

This repo is meant to be **traversed, not read front-to-back.** Start at the entry that fits you:

| You want to… | Go to |
|---|---|
| **Run it** | [docs/usage.md](docs/usage.md) → [SKILL.md](SKILL.md) |
| Understand **why it exists** | [docs/why.md](docs/why.md) |
| See **who it's for / what you get** | [docs/who-its-for.md](docs/who-its-for.md) |
| Read the **ethics & authorization** rules | [docs/ethics.md](docs/ethics.md) ⛔ read before active testing |
| Browse the **playbooks** (the tree) | [references/00-map.md](references/00-map.md) (signal→file jump table) |
| Find the **frameworks & tool URLs** | [RESEARCH.md](RESEARCH.md) (bibliography hub → per-domain `research/`) |

## Repository layout

```
Longinus/
├── README.md                       ← English landing (you are here)
├── README.ko.md                    ← Korean landing
├── CHANGELOG.md                    ← version history (all releases)
├── SKILL.md                        ← the orchestration brain (the entry point)
├── RESEARCH.md                     ← bibliography hub (indexes the research/ tree)
├── agents/                         ← OPTIONAL multi-agent layer (orchestrator + Blue + 5 Red specialists)
├── docs/                           ← the concept, split into linked docs
│   ├── why.md                      ← why this exists (the 3 trends)
│   ├── who-its-for.md              ← audiences + what you get (a report, not a scanner dump)
│   ├── ethics.md                   ← ethics & authorization summary
│   └── usage.md                    ← how to use it / read it as a knowledge base
├── research/                       ← per-domain bibliography (frameworks + canonical tool URLs)
│   ├── rationale.md  recon.md  web.md  api.md  identity.md
│   ├── secrets-supply-chain.md  ai-llm.md  cloud-infra.md  mobile.md
│   └── ctf.md  process.md  meta-resources.md
└── references/                     ← the domain tree (the playbooks)
    ├── 00-map.md                   ← master navigable tree + signal→file jump table
    ├── authorization-and-scope.md                            ← ⛔ authorization gate
    ├── pattern-triggers.md                                   ← 🎯 attacker principles + code pattern → vulnerability lookup
    ├── limitations.md                                        ← ⚠️ what this skill cannot find (honest limits)
    ├── methodology.md  chaining-and-impact.md                ← lifecycle + 🔗 compose findings → impact
    ├── severity-and-triage.md  reporting-and-disclosure.md   ← the governance spine
    ├── recon/  web/  api/  identity/  secrets-and-supply-chain/
    ├── ai-llm/  cloud-and-infra/  mobile/
    └── binary-exploitation/  reverse-engineering/  cryptography/  forensics/  tooling/
```

The tree mirrors how professional offense is organized: a shared **methodology + governance** spine,
then **specialist branches** you descend into based on the target. Each playbook leaf links down to its
matching `research/<domain>.md` for the canonical frameworks and tool URLs.

## Status

The domain leaves and `research/` bibliography are living documents; extend them as techniques
evolve (policy: [research/meta-resources.md](research/meta-resources.md)).

**Changelog** — the full version history now lives in [CHANGELOG.md](CHANGELOG.md).
