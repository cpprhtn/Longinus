# 🔗 Remote-code-execution chains

The top prize: turn an input bug into **arbitrary code on the server**. RCE is usually a chain — a
primitive (write/inject/deserialize) composed with a way to *execute* it. Parent:
[../chaining-and-impact.md](../chaining-and-impact.md).

## Chain 1 — arbitrary upload + path/exec → webshell

1. **Unrestricted file upload** (no content-type/extension check) → [../web/file-upload-and-path.md](../web/file-upload-and-path.md).
2. Land it **inside the webroot** (or use **path traversal** to choose the path) with an executable
   extension the server will run (`.php`, `.jsp`, `.aspx`).
3. Request the uploaded file → **webshell → RCE**.

## Chain 2 — SQLi → file write / stacked command → RCE

1. **SQL injection** with sufficient DB privilege → [../web/injection.md](../web/injection.md).
2. Escalate: MySQL `INTO OUTFILE` a webshell into the webroot; MSSQL `xp_cmdshell`; PostgreSQL
   `COPY ... TO PROGRAM`; or read creds and pivot → **RCE / lateral**.

## Chain 3 — insecure deserialization → gadget → RCE

1. The app deserializes attacker-controlled bytes (Java `rO0`, PHP `O:`, Python `pickle`, .NET,
   `ViewState`, **a model file `torch.load`**) → [../web/deserialization.md](../web/deserialization.md) ·
   [../ai-llm/model-supply-chain.md](../ai-llm/model-supply-chain.md).
2. Supply a **gadget chain** (ysoserial / PHPGGC / a `__reduce__`) → **RCE on load**.

## Chain 4 — prototype pollution → sink gadget → RCE

1. **Prototype pollution** via a deep-merge/`Object.assign` of attacker JSON (`__proto__`) →
   [../web/prototype-pollution.md](../web/prototype-pollution.md).
2. Pollute a property a downstream **gadget** trusts (template options, `child_process` opts, config) →
   **RCE** (server-side) or auth-bypass.

## Chain 5 — reachable dependency CVE / SSTI

- A known-CVE or **hallucinated** dependency that reaches a vulnerable sink → [../secrets-and-supply-chain/dependency-supply-chain.md](../secrets-and-supply-chain/dependency-supply-chain.md).
- **Server-side template injection** (`{{7*7}}`→`49`) → sandbox escape → RCE → [../web/injection.md](../web/injection.md).

## Real-world cases

**Log4Shell** (CVE-2021-44228, JNDI lookup → RCE) · **Apache Commons-Collections** Java-deser (FoxGlove,
2015) · [TOP RCE (HackerOne)](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPRCE.md).
See the per-leaf "Real-world cases" footers.

## Prove each link

A benign marker (`id`, a DNS callback, `touch /tmp/<canary>`) proves execution — **never** drop a real
shell or persist on a target you don't own ([../authorization-and-scope.md](../authorization-and-scope.md)).

## 🛡️ Defensive seal

- **Upload:** store outside webroot, random names, validate content server-side, never execute uploads (Chain 1).
- **Parameterized queries + least-privilege DB user** (no `FILE`/`xp_cmdshell`) (Chain 2).
- **Never deserialize untrusted input; allowlist types; `safetensors`/`weights_only=True`** (Chain 3).
- **Reject `__proto__`/`constructor` keys; `Object.create(null)`** (Chain 4).
- **SCA gate + reachable-CVE check; sandbox/parameterize templates** (Chain 5).

→ [../enforce-forward.md](../enforce-forward.md) · [../templates/validation-layer.md](../templates/validation-layer.md) · [../templates/ci-gates.md](../templates/ci-gates.md).

Back to [chain catalog](../chaining-and-impact.md) · [tree](../00-map.md).
