# 🔗 AI / agent attack chains

The 2025–26 frontier: the model is a **confused deputy** that turns *text* into *actions*. The chains
below compose an injection with the agent's real privileges. Parent:
[../chaining-and-impact.md](../chaining-and-impact.md).

## Chain 1 — the *lethal trifecta* (indirect injection → tool → exfil)

The killer combination (Simon Willison) of **three** properties in one agent:
1. **Access to private data** (the user's mail, files, tokens, RAG store).
2. **Exposure to untrusted content** — a web page it browses, a PDF/email/ticket it summarizes, a RAG
   document → carries a hidden instruction → [../ai-llm/prompt-injection.md](../ai-llm/prompt-injection.md).
3. **An exfiltration channel** — a send-email/HTTP/markdown-image tool, or just rendering a URL.

Inject in (2) → the model reads private data (1) → ships it out (3). No "hacking the model" needed —
the *plumbing* is the vuln → [../ai-llm/agentic-and-mcp.md](../ai-llm/agentic-and-mcp.md).

## Chain 2 — injection → tool argument → classic web sink

1. Direct/indirect injection makes the agent call a tool with **attacker-chosen arguments**.
2. The tool argument is an unguarded **web sink**: a URL tool → **SSRF** ([../web/ssrf.md](../web/ssrf.md));
   a DB tool → **SQLi** ([../web/injection.md](../web/injection.md)); a shell/code tool → **RCE**. The
   model is just the new caller of the same old sink.

## Chain 3 — model output → unsafe rendering → XSS → ATO

1. The model's reply is rendered as **HTML/markdown** without encoding → emit
   `<img src=x onerror=...>` → **XSS** → [../web/xss.md](../web/xss.md).
2. Steal the session → **ATO** ([account-takeover.md](account-takeover.md)). "The model said a weird
   thing" becomes a real exploit.

## Chain 4 — supply chain: poisoned model / RAG

- A **malicious model file** (`torch.load`/pickle/Keras-Lambda) → **RCE on load** →
  [../ai-llm/model-supply-chain.md](../ai-llm/model-supply-chain.md).
- **RAG/data poisoning**: attacker-controlled docs enter the index → bias/backdoor answers or inject at
  retrieval time → [../ai-llm/prompt-injection.md](../ai-llm/prompt-injection.md).
- **MCP cross-tool / tool-poisoning**: a malicious MCP server's tool description injects instructions, or
  output of tool A feeds tool B → [../ai-llm/agentic-and-mcp.md](../ai-llm/agentic-and-mcp.md).

## Real-world cases

Johann Rehberger / **EmbraceTheRed** indirect-injection exfil (ChatGPT/Gemini memory, markdown-image
data theft) up to **GitHub Copilot RCE (CVE-2025-53773)** · [Simon Willison: lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)
· [embracethered.com/blog](https://embracethered.com/blog/).

## Prove each link

A benign marker (the agent emails *your* canary address, fetches *your* collaborator URL, renders
`alert(document.domain)`) proves the action — never exfiltrate real third-party data
([../authorization-and-scope.md](../authorization-and-scope.md)).

## 🛡️ Defensive seal

- **Break the trifecta:** remove *one* leg — least-privilege tools, no untrusted content in a
  data-accessing agent, or no unrestricted exfil tool.
- **Validate/encode model output into every sink** exactly like raw user input (Chains 2–3).
- **Human-in-the-loop for high-impact tool calls; deterministic tool-arg allowlists; tenant isolation on RAG.**
- **`safetensors` + a `modelscan` gate; vet MCP servers like dependencies** (Chain 4).

→ [../enforce-forward.md](../enforce-forward.md) (AI row) · [../templates/validation-layer.md](../templates/validation-layer.md) · [../templates/ci-gates.md](../templates/ci-gates.md).

Back to [chain catalog](../chaining-and-impact.md) · [tree](../00-map.md).
