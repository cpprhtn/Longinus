# Agentic risks, tools/MCP & RAG/vector weaknesses (LLM06/08/10)

When an LLM stops just *talking* and starts *acting* — calling tools/functions, querying databases,
browsing, executing code, spending money, orchestrating other agents — prompt injection becomes
*action* injection. This leaf covers **Excessive Agency (LLM06)**, **Vector & Embedding Weaknesses
(LLM08)**, and **Unbounded Consumption (LLM10)**, plus MCP and multi-agent chains.

## Excessive Agency (LLM06) — the core agentic risk

Three knobs, each a finding when set too high:
- **Excessive permissions** — the agent's tools/credentials can do more than the use case needs
  (a "read my calendar" agent that holds full account write access; a DB tool with a read-write admin
  connection).
- **Excessive functionality** — tools exposed "just in case" (a shell tool, an arbitrary-HTTP tool, a
  send-email tool) that an injection can repurpose.
- **Excessive autonomy** — the agent executes high-impact actions (delete, pay, deploy, message
  externally) **without human approval**.

**Test:**
1. Enumerate every tool/function the agent can call and the **real privilege** behind each (what
   credential/scope does the tool use? what can it reach?).
2. Via direct or indirect injection ([prompt-injection.md](prompt-injection.md)), try to make the agent
   **invoke a sensitive tool with attacker-chosen arguments** — exfiltrate data, send a message, write
   a file, run code, make a purchase, change a setting.
3. Check whether sensitive actions require **confirmation/human-in-the-loop** or just happen.
4. Test **argument injection** into tools: can you control a tool's parameters to reach SSRF (URL
   tool), SQLi (DB tool), path traversal (file tool), or command injection (shell/code tool)? The tool
   boundary is a classic [../web/](../web/README.md) sink — the model is just the new caller.

**Fix:**
- **Least privilege per tool:** minimal scopes, scoped/short-lived credentials, read-only where
  possible, per-tenant isolation. The agent's effective permissions = the union of its tools'
  permissions — minimize it.
- **Least functionality:** expose only the tools the task needs; no general shell/HTTP/eval tools
  unless essential and sandboxed.
- **Human-in-the-loop** for high-impact/irreversible actions; out-of-band confirmation for money/data/
  external comms.
- **Validate tool arguments deterministically** (allowlists, type/range checks, the same sink defenses
  as web) — never pass model-chosen args straight to a powerful sink.
- **Authorize actions in code**, tied to the *end user's* permissions, not the agent's blanket
  identity (a confused-deputy fix).

## MCP & multi-agent chains

Model Context Protocol servers and agent-to-agent setups expand the surface:
- **Untrusted MCP servers / tools** are code you're trusting — vet them like dependencies
  ([../secrets-and-supply-chain/dependency-supply-chain.md](../secrets-and-supply-chain/dependency-supply-chain.md));
  a malicious tool can return injected instructions or exfiltrate the context.
- **Tool-description / "tool poisoning" injection:** instructions hidden in a tool's description/
  metadata that the model reads → influence behavior. Inspect what the model actually ingests about
  each tool.
- **Cross-tool / cross-agent trust:** output of agent/tool A becomes input to B — an injection in A
  propagates. Treat every inter-agent message as untrusted.
- **Credential/secret handling** in MCP configs (tokens in plaintext config — see
  [../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md)).
- **Confused-deputy across the chain:** ensure each hop re-checks authorization against the real user,
  not the agent's elevated identity.

## RAG & vector/embedding weaknesses (LLM08)

RAG adds a retrieval store full of (often user/tenant) data into the context — new leakage and
injection paths:
- **Cross-tenant / over-broad retrieval:** does the vector search filter by the requesting user/tenant
  *before* retrieval? If not, user A's query can surface user B's documents (an "embedding IDOR").
  **Test:** as user A, craft queries to retrieve B's content; confirm isolation. **Fix:** enforce
  per-tenant filters/namespaces at query time, server-side — not via the prompt.
- **Indirect injection via documents:** poisoned content in the knowledge base injects instructions
  when retrieved ([prompt-injection.md](prompt-injection.md)). **Fix:** treat retrieved text as
  untrusted; isolate it; don't let it drive tool calls unchecked.
- **Embedding inversion / data leakage:** sensitive data recoverable from stored embeddings; access to
  the vector DB = access to the data. **Fix:** protect the vector store like a primary datastore
  (authn/z, encryption, least privilege); minimize sensitive content indexed.
- **Data/knowledge poisoning (LLM04):** attacker-controlled docs entering the index bias or backdoor
  answers. **Fix:** vet ingestion sources, provenance, and review pipelines.

## Unbounded Consumption (LLM10)

LLM calls cost money and compute — abuse = DoS + bill shock + IP theft:
- **Test:** unthrottled/expensive prompts, huge inputs/outputs, recursive agent loops, fan-out tool
  calls, and **model extraction** (mass querying to clone behavior/data).
- **Fix:** per-user rate/token/cost limits and budgets, input/output size caps, loop/step ceilings and
  timeouts for agents, monitoring/alerting on spend and anomalous usage, and abuse detection.

## Vulnerable ↔ fixed application code

### Excessive agency — unvalidated tool arguments

```python
# ❌ VULNERABLE — model controls DB query with no validation
tools = [{"name": "run_query", "description": "Run a database query",
          "parameters": {"query": {"type": "string"}}}]

def run_query(query):
    return db.execute(query)   # model sends "DROP TABLE users" if prompt-injected

# ✅ FIXED — validate tool arguments deterministically
ALLOWED_TABLES = {'products', 'categories', 'reviews'}

def run_query(query):
    parsed = sqlparse.parse(query)[0]
    if parsed.get_type() != 'SELECT':
        raise PermissionError("Only SELECT queries allowed")
    tables = extract_tables(parsed)
    if not all(t in ALLOWED_TABLES for t in tables):
        raise PermissionError("Query references unauthorized table")
    return db.execute(query)
```

### Cross-tenant RAG leakage

```python
# ❌ VULNERABLE — no tenant filter on vector search
def answer(user_question, user):
    results = vector_store.similarity_search(user_question, k=5)
    # returns documents from ALL tenants — user A reads tenant B's data

# ✅ FIXED — enforce tenant isolation at query time (server-side, not prompt-based)
def answer(user_question, user):
    results = vector_store.similarity_search(
        user_question,
        k=5,
        filter={"tenant_id": user.tenant_id}   # hard filter, not a prompt instruction
    )
```

### Missing human-in-the-loop for sensitive actions

```python
# ❌ VULNERABLE — agent auto-executes payment without approval
async def handle_tool_call(tool_name, args):
    if tool_name == "send_payment":
        return payment_api.send(args["to"], args["amount"])  # injection → money gone

# ✅ FIXED — gate sensitive actions on explicit user approval
SENSITIVE_TOOLS = {"send_payment", "delete_account", "send_email"}

async def handle_tool_call(tool_name, args, user_session):
    if tool_name in SENSITIVE_TOOLS:
        approval = await request_user_approval(
            user_session,
            f"Agent wants to {tool_name} with args: {args}. Approve?"
        )
        if not approval:
            return {"error": "User denied this action"}
    return execute_tool(tool_name, args)
```

---

## Confirm (PoC)

Show an unauthorized *action* or *leak*, reproducibly: "an injected calendar invite causes the agent
to email my data to an external address with no approval", "user A's query retrieves user B's RAG
document", or "an unbounded agent loop drives N tool calls / $X cost". Benign markers only; don't
actually exfiltrate real third-party data.

## CTF / red-team angle

Agent challenges: trick the agent into calling a "win"/flag tool, chain an indirect injection through a
document into a tool call, or exploit a tool's argument as a classic web sink (SSRF/SQLi/RCE). Always
enumerate the tools and their real privileges first.

## References

[OWASP LLM06/08/10:2025, LLM04](https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/) ·
[MITRE ATLAS](https://atlas.mitre.org/) ·
[OWASP Agentic Security Initiative](https://genai.owasp.org/) ·
[MCP security](https://modelcontextprotocol.io/) · [NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework).
Full bibliography: [research/ai-llm.md](../../research/ai-llm.md). Back to [ai-llm/](README.md).
