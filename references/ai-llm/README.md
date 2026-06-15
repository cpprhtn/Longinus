# 🤖 ai-llm/ — LLM, agent & RAG application security

If your vibe-coded app calls an LLM — a chatbot, a "summarize this", a RAG assistant, an autonomous
agent, an MCP toolchain — it has a new, weird attack surface that classic web testing misses. This
branch maps the **OWASP Top 10 for LLM Applications 2025** and the agentic threats, and shows how to
red-team an AI feature. References: OWASP GenAI/LLM Top 10:2025, MITRE ATLAS, NIST AI RMF.

> ⛔ Gate: red-team only your own AI feature, an authorized program's, or a lab. Test for *security*
> outcomes (data exfil, unauthorized actions, prompt/secret leakage) — don't generate harmful content
> as the goal.
>
> The classic web branch still applies: an LLM app is also a web app/API. Test
> [../web/](../web/README.md) and [../secrets-and-supply-chain/](../secrets-and-supply-chain/README.md)
> too. The model's output is just another untrusted input to the rest of your system.

## ⏱️ First 5 minutes — AI/LLM quick checks

Run these before descending into the leaves below.

1. Find all model loading calls → `trust_remote_code=True`? pickle-based? unpinned model refs?
2. Enumerate all tool/function definitions → what real permissions does each have?
3. Trace where model output goes → does it reach `innerHTML` / SQL / shell / `eval` / URL fetch?
4. Find RAG/vector queries → is there a tenant/user filter at query time?
5. Read the system prompt → does it contain secrets or sensitive logic?

## OWASP Top 10 for LLM Applications 2025

| ID | Risk | Where / how |
|---|---|---|
| **LLM01** | **Prompt Injection** (direct & indirect) | [prompt-injection.md](prompt-injection.md) — the #1 LLM risk |
| **LLM02** | Sensitive Information Disclosure | model leaks PII/secrets/training data/other users' context; see below + [prompt-injection.md](prompt-injection.md) |
| **LLM03** | Supply Chain | malicious model file (pickle/`.pt`/Keras) = RCE on load, hub namesquatting, vulnerable ML libs | [model-supply-chain.md](model-supply-chain.md) |
| **LLM04** | Data & Model Poisoning | tainted training/fine-tune/RAG data → backdoors/bias/triggers | [model-supply-chain.md](model-supply-chain.md) |
| **LLM05** | Improper Output Handling | trusting model output → XSS/SQLi/SSRF/RCE downstream; see below |
| **LLM06** | Excessive Agency | agent has too much permission/autonomy/tool access | [agentic-and-mcp.md](agentic-and-mcp.md) |
| **LLM07** | System Prompt Leakage | system prompt (and secrets/rules in it) extracted | [prompt-injection.md](prompt-injection.md) |
| **LLM08** | Vector & Embedding Weaknesses | RAG/embedding leakage, cross-tenant retrieval, injection via documents | [agentic-and-mcp.md](agentic-and-mcp.md) |
| **LLM09** | Misinformation | hallucinations relied on for security/safety decisions |
| **LLM10** | Unbounded Consumption | token/cost/DoS, model extraction via mass querying |

## Branch map

| Leaf | Covers |
|---|---|
| [prompt-injection.md](prompt-injection.md) | LLM01/07/02: direct & indirect injection, jailbreaks, system-prompt & data exfiltration, output handling (LLM05) |
| [agentic-and-mcp.md](agentic-and-mcp.md) | LLM06/08: excessive agency, tool/function-call abuse, MCP & multi-agent chains, RAG/vector store leakage, unbounded consumption |
| [model-supply-chain.md](model-supply-chain.md) | LLM03/04: malicious model files (pickle/`.pt`/Keras → RCE), hub namesquatting, `trust_remote_code`, data/weight poisoning & backdoors |

## How to red-team an AI feature (workflow)

1. **Map the AI pipeline.** Where's the system prompt? What untrusted data enters the context
   (user input, retrieved docs, web pages, emails, tool outputs, other users' data)? What can the
   model *do* (tools/functions, DB queries, code exec, send messages, spend money)? Where does its
   output *go* (rendered as HTML? fed to `eval`/SQL/a shell? to another agent)? **What models/weights
   does it load, from where, and with which loader?** ([model-supply-chain.md](model-supply-chain.md))
2. **Trust-boundary the context window.** Everything in the context is effectively "instructions" to
   the model — so any untrusted text in it is an injection vector. List every source.
3. **Attack each boundary** with the leaf playbooks: direct injection (user), indirect injection
   (documents/tools/web), output-handling sinks, agency/permission abuse, data leakage.
4. **Check the non-AI controls** — because prompt injection is *not fully solvable at the prompt
   level*, the real defenses are deterministic: least-privilege tools, human approval for sensitive
   actions, output validation, tenant isolation, rate/cost limits. Test that those exist.
5. **Report** with the same rigor (reproducible prompt, observed unauthorized outcome, fix).

## The core mental model

> An LLM cannot reliably distinguish "instructions from the developer" from "instructions hidden in the
> data it's processing." Treat the model as a **confused deputy**: it will faithfully do what the most
> persuasive text in its context says. Therefore **security must live around the model, not inside the
> prompt.** Guardrails in the system prompt are speed bumps, not boundaries.

## CTF / red-team angle

AI-category CTF challenges: extract a hidden flag from the system prompt (LLM07), bypass a guardrail to
make the model reveal/secret-do something (LLM01), indirect injection via an uploaded document, or
exploit `Improper Output Handling` where the model's reply is rendered as HTML (→ XSS) or run as a
query (→ SQLi). Map the pipeline, then inject at the weakest boundary.

## References

[OWASP Top 10 for LLM Applications 2025](https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/) ·
[MITRE ATLAS](https://atlas.mitre.org/) · [NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework) ·
[OWASP Agentic Security Initiative](https://genai.owasp.org/) · [MCP](https://modelcontextprotocol.io/).
Full bibliography: [research/ai-llm.md](../../research/ai-llm.md). Back to [tree](../00-map.md).
