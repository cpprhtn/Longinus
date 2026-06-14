# How to use Longinus

↑ back to [README](../README.md) · doc map: [why](why.md) · [who-its-for](who-its-for.md) · [ethics](ethics.md)

## Install

Longinus is a Claude Code Skill — Claude Code auto-discovers any skill placed in a `skills/` directory:

```bash
# Per-project (committed with your repo, shared with the team)
git clone https://github.com/cpprhtn/Longinus.git .claude/skills/longinus

# Personal (available across all your projects)
git clone https://github.com/cpprhtn/Longinus.git ~/.claude/skills/longinus
```

Clone the **whole tree** so `SKILL.md` sits at the folder root with `references/` and `research/`
beside it — the skill's internal links are relative, so copying individual files breaks it. A new
Claude Code session picks it up automatically; then ask a security question (or run `/longinus`).

It's **stack-agnostic** — Python/FastAPI, Node/Express, React Native, Go, Terraform/cloud, an LLM/RAG
app — because it **profiles the target first** ([SKILL.md](../SKILL.md) Step 1), then routes to the
matching branch. It guides Claude to audit your code (read-only by default); it is *not* an automatic
scanner that runs on its own.

## Just ask

This is a skill, so you mostly just *ask*:

```
"Longinus my repo before I launch."
"Run a security audit on this Express API."
"Red-team the prompt injection surface of my RAG agent."
"Here's my bug bounty scope for acme.com — help me work it."   (with authorization)
"Walk me through the crypto category for this CTF challenge."
```

The skill will: gate on authorization → profile the target → pick the right branches of the tree →
run the playbooks (read-only by default) → triage → report. The orchestration logic lives in
[SKILL.md](../SKILL.md).

## Read it as a knowledge base

You can also traverse the tree directly:

- **Playbooks:** start at [references/00-map.md](../references/00-map.md) — a signal→file jump table
  routes you to the one leaf you need. Don't read it all; traverse it.
- **Bibliography:** [RESEARCH.md](../RESEARCH.md) is the index to the per-domain `research/` files
  (frameworks + canonical tool URLs). Each playbook leaf links to its matching `research/<domain>.md`.

> Before any **active** testing, clear the gate in [ethics.md](ethics.md) /
> [references/authorization-and-scope.md](../references/authorization-and-scope.md).
