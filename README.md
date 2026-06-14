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
- **CTF players & learners** drilling a specific category in depth.

## What makes it different

Not just another playbook pack:

- **Reasons from principles, not a checklist.** Instead of walking a fixed list, it derives bugs from
  six root principles that *generate* vulnerability classes — trust boundaries, parser differentials,
  confused deputy, state & time, encoding, the developer's unstated assumptions →
  [the attacker's mindset](references/attacker-mindset.md). It catches classes that don't even have a
  name yet.
- **Rates findings by chained impact, not in isolation.** Issues that are individually "low" can
  compose into account takeover; that chain is treated as one **Critical** →
  [chaining](references/chaining-and-impact.md).
- **Ruthlessly suppresses false positives.** No reproducible PoC, no confirmed finding. An LLM's
  weakest point is the confident hallucinated bug, so a low false-positive rate is what makes the
  output trustworthy → [severity & triage](references/severity-and-triage.md).

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
    ├── authorization-and-scope.md  attacker-mindset.md       ← ⛔ gate + 🧠 the generative lens
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

`v0.1.0` — initial tree. The domain leaves and the `research/` bibliography are living documents;
extend them as techniques evolve (policy: [research/meta-resources.md](research/meta-resources.md)).
