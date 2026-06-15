# 🕸️ Longinus agents — optional multi-agent layer

The [Claude Code Skill](../SKILL.md) works **standalone** — you don't need these. This folder is an
**optional force-multiplier**: a set of [subagents](https://code.claude.com/docs/en/sub-agents) that run
a Longinus audit as a coordinated team, each specialist in **its own context window**.

## Why use it

A single-context audit suffers **session contamination** — findings from one domain (or a previous
target) bias the next ([../references/limitations.md](../references/limitations.md)). Subagents fix this
*structurally*: each specialist audits its domain in an isolated context and returns only its findings,
so there's no cross-domain confirmation bias, and domains can run **in parallel**.

The team mirrors the tree: an **orchestrator** profiles the target, gates authorization, dispatches the
relevant domain specialists, then de-dups → chains → triages → writes one ranked report. Offense drives,
defense seals.

## Install

These are **not** auto-loaded from the skill folder. Copy them where Claude Code scans for agents:

```bash
# Personal (all your projects)
cp agents/*.md ~/.claude/agents/

# Or per-project (commit with your repo, share with the team)
mkdir -p .claude/agents && cp agents/*.md .claude/agents/
```

Restart the session (or run `/agents`) to load them. They **preload the `longinus` skill**
(`skills: longinus` frontmatter), so the Longinus skill must also be installed
([../README.md](../README.md) → Install). Then just ask:

> "Run a Longinus audit on this repo."  →  the **longinus-orchestrator** takes over.

## The roster

| Agent | Role | Tools |
|---|---|---|
| **longinus-orchestrator** | gate → profile → dispatch → chain → triage → report | all (delegates) |
| **longinus-secrets** | secrets & dependency/supply-chain (always runs first) | read-only + Bash |
| **longinus-web** | OWASP Top 10:2025 web classes | read-only + Bash |
| **longinus-api-identity** | API authz (BOLA/BFLA) + OAuth/JWT/SAML/MFA | read-only + Bash |
| **longinus-cloud** | cloud / IaC / containers (CIS) | read-only + Bash |
| **longinus-ai** | LLM/agent/RAG (prompt injection, agency, model supply chain) | read-only + Bash |

Specialists are **read-only on your code** (no Edit/Write) — they audit, they don't change things, by
design. Each carries the Longinus discipline: ⛔ authorization gate · prove-it-or-park-it · two lenses.

## Future: plugin packaging

To ship the skill + agents as one install, this repo can be packaged as a Claude Code
[plugin](https://code.claude.com/docs/en/plugins) (a manifest bundling `skills/` + `agents/`). The
standalone skill + optional-agents layout here is the portable baseline; the plugin is an additive path.

Back to [README](../README.md) · [SKILL.md](../SKILL.md) · [tree](../references/00-map.md).
