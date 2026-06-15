# 🤖 ai-llm — references & tooling

Bibliography for the [`references/ai-llm/`](../references/ai-llm/README.md) branch (OWASP LLM Top
10:2025, agentic/MCP). ↑ back to the [RESEARCH hub](../RESEARCH.md).

## Frameworks & standards

- **OWASP Top 10 for LLM Applications 2025** — LLM01 Prompt Injection · LLM02 Sensitive Information
  Disclosure · LLM03 Supply Chain · LLM04 Data & Model Poisoning · LLM05 Improper Output Handling ·
  LLM06 Excessive Agency · LLM07 System Prompt Leakage · LLM08 Vector & Embedding Weaknesses ·
  LLM09 Misinformation · LLM10 Unbounded Consumption. →
  https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/
- **MITRE ATLAS** — adversarial ML threat matrix (analog of ATT&CK for AI). → https://atlas.mitre.org/
- **NIST AI RMF** (AI 100-1) + Generative AI Profile (NIST AI 600-1). →
  https://www.nist.gov/itl/ai-risk-management-framework
- **OWASP GenAI Security Project** (incl. Agentic Security Initiative). → https://genai.owasp.org/
- **Model Context Protocol (MCP)** spec + security considerations. → https://modelcontextprotocol.io/

## Tooling

> Manual injection testing stays primary — these assist, they don't replace understanding the pipeline.

- **Prompt/agent red-team:** [garak](https://github.com/NVIDIA/garak) (LLM vuln scanner) ·
  [promptfoo](https://www.promptfoo.dev/) (eval/red-team) · [PyRIT](https://github.com/Azure/PyRIT) ·
  [deepteam](https://github.com/confident-ai/deepteam) · [Giskard](https://www.giskard.ai/).
- **Model supply chain (LLM03):** [modelscan](https://github.com/protectai/modelscan) &
  [picklescan](https://github.com/mmaitre314/picklescan) (flag dangerous pickle opcodes) ·
  [fickling](https://github.com/trailofbits/fickling) (pickle decompile/inject/check) ·
  prefer [safetensors](https://github.com/huggingface/safetensors); pin + verify model provenance like
  dependencies. [HuggingFace Hub security](https://huggingface.co/docs/hub/en/security).
- Map findings to OWASP LLM Top 10:2025 / MITRE ATLAS.

## Further reading (curated lists)

- [corca-ai/awesome-llm-security](https://github.com/corca-ai/awesome-llm-security) — tools, papers &
  projects on LLM security.
- [wearetyomsmnv/Awesome-LLMSecOps](https://github.com/wearetyomsmnv/Awesome-LLMSecOps) — LLM/agentic
  security operations.
- [ucsb-mlsec/Awesome-Agent-Security](https://github.com/ucsb-mlsec/Awesome-Agent-Security) +
  [LLMSecurity/awesome-agent-skills-security](https://github.com/LLMSecurity/awesome-agent-skills-security)
  — agent / tool-use / skill-ecosystem attacks & defenses.
- [gmh5225/awesome-ai-security](https://github.com/gmh5225/awesome-ai-security) — AI security for
  pentesters/bug-hunters.

## Related references

An LLM app is also a web app/API: [web.md](web.md) (output → XSS/SQLi/SSRF sinks),
[api.md](api.md), [secrets-supply-chain.md](secrets-supply-chain.md) (LLM03 supply chain) ·
[process.md](process.md) · [meta-resources.md](meta-resources.md).
