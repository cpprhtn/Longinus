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

## Why

- **AI-generated code ships fast and insecure.** You need a systematic way to audit it before launch.
- **Security knowledge is scattered.** Instead of hunting through OWASP, bug-bounty writeups, and CTF
  lore every time, follow one proven methodology.
- **Scanners drown you in noise.** Longinus follows a *prove-it-or-park-it* rule and reports only what
  it can actually demonstrate.

## Who it's for

- **Developers & founders** auditing their own (often AI-written) code before launch.
- **Bug-bounty hunters & pentesters** who need a repeatable methodology against authorized scope.

## What makes it different

Not just another playbook pack:

- **Three audit modes (quick / standard / deep).** Quick mode runs mechanical grep-based scans with
  fixed severities — ideal for CI gates and small on-device models (11B+). Standard mode is a full
  single-target audit. Deep mode sweeps all domains with cross-domain chaining analysis.
- **Reasons from principles, not a checklist.** Instead of walking a fixed list, it derives bugs from
  six root principles that *generate* vulnerability classes — trust boundaries, parser differentials,
  confused deputy, state & time, encoding, the developer's unstated assumptions →
  [pattern triggers & attacker principles](references/pattern-triggers.md). It catches classes that
  don't even have a name yet.
- **Rates findings by chained impact, not in isolation.** Issues that are individually "low" can
  compose into account takeover; that chain is treated as one **Critical** →
  [chaining](references/chaining-and-impact.md).
- **Ruthlessly suppresses false positives.** No reproducible PoC, no confirmed finding. Mandatory
  severity gates force Critical/High findings *down* when preconditions aren't met — an LLM's weakest
  point is severity inflation → [severity & triage](references/severity-and-triage.md).
- **Integrated dependency health checks.** SCA tools run as part of the audit flow; CVEs without
  confirmed reachability are flagged Info-only, not inflated to High/Critical.
- **Mechanical pattern lookup for code audits.** A
  [pattern-trigger table](references/pattern-triggers.md) maps code patterns directly to
  vulnerability checks with false-positive guards — no principle synthesis needed.
- **Honest about its limits.** A [limitations doc](references/limitations.md) says what static/LLM
  analysis *cannot* find, so a clean report never implies "no vulnerabilities exist."

## Audit modes

| Mode | When to use | What happens |
|---|---|---|
| **quick** | Fast first pass, CI gate checks, **small on-device models (11B+)** | Runs mechanical grep scans from each leaf's `## Mechanical scan` section. Fixed severities, no CVSS scoring, no chaining analysis. One table of findings. |
| **standard** | Default. Full audit for one target. | Profile → route → full leaf analysis → PoC → triage (CVSS 4.0) → report. |
| **deep** | Comprehensive formal audit, multi-domain sweep. | Standard + all secondary domain quick-checks + cross-domain chaining ([chaining-and-impact.md](references/chaining-and-impact.md)). |

**Quick mode is designed for small on-device models** (e.g., Qwen 3 8B, Llama 3.1 8B) that can
follow mechanical instructions but struggle with open-ended reasoning. It works entirely through
pattern matching — no principle synthesis, no CVSS calculation, no chain analysis. If you're running
a local model for security checks, start here.

Specify the mode when invoking: *"run a quick security audit"*, *"deep audit this repo"*, or just
ask a security question (defaults to **standard**).

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

## Usage

After install, just ask a security question, or run `/longinus`:

> "Audit this repo before I launch." · "Review the security of this FastAPI API." · "Check this React
> Native app too."

**It's stack-agnostic** — Python/FastAPI, Node/Express, React Native, Go, Terraform/cloud, an LLM/RAG
app — because it **profiles the target first**, then routes to the right playbooks. It guides Claude to
audit your code (read-only by default); it is *not* an automatic scanner that runs on its own. Before
testing anything you don't own, clear the [authorization gate](references/authorization-and-scope.md).

> ⛔ **Offense in service of defense.** Longinus is for code you own, targets you're explicitly
> authorized to test, and CTF/lab environments — never for unauthorized systems. See
> [docs/ethics.md](docs/ethics.md).

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
├── SKILL.md                        ← the orchestration brain (the entry point)
├── RESEARCH.md                     ← bibliography hub (indexes the research/ tree)
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
