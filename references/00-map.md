# 🗺️ Longinus domain tree (master map)

This is the navigable index of the whole skill. The spine (governance) is shared by every engagement;
the branches are specialist domains you descend into based on the target. Each leaf is a self-contained
playbook: *what the weakness is · how to find it · how to confirm it · how to fix it.*

> **Traverse, don't read it all.** From a target type or a concrete signal, jump to **one** leaf and
> open only that file. Go deeper only when the leaf points you deeper. Pull spine docs in when that
> phase arrives. Use [SKILL.md](../SKILL.md) for orchestration; ⛔ pass the
> [authorization gate](authorization-and-scope.md) before any active testing.

---

## ⚡ Fast jump: signal → exact file (one hop)

You already see something specific — go straight to the leaf, skip the branch READMEs. (Right column
is the *only* file you need to open for that signal.)

> **For source-code audits** where you're scanning code patterns rather than observing runtime
> signals, use the [pattern-catalog.md](pattern-catalog.md) lookup table instead — it maps code
> patterns directly to vulnerability checks with false-positive guards.
> For honest capability limits, see [limitations.md](limitations.md).

| What you're looking at / the signal | Open only this |
|---|---|
| Unsure where to start in a pre-launch/vibe-coded repo; want the cheap high-yield pass | [basic-vibe-triage.md](basic-vibe-triage.md) |
| `id`/UUID/filename in the URL or body; can I read *another user's* object? | [web/access-control.md](web/access-control.md) |
| Admin/privileged action reachable by a normal/no-auth user | [web/access-control.md](web/access-control.md) · [api/bola-bfla.md](api/bola-bfla.md) |
| Input lands in SQL / a shell / a template / XML parser | [web/injection.md](web/injection.md) |
| My input is reflected into the HTML/JS of the page | [web/xss.md](web/xss.md) |
| A param/feature where the **server fetches a URL** (webhook, import, preview, PDF) | [web/ssrf.md](web/ssrf.md) |
| Login / password reset / session cookie / brute-force | [web/auth-and-session.md](web/auth-and-session.md) |
| Default/seeded creds (`admin/admin`, `admin/1234`), hard-coded login branch, debug login-as | [web/auth-and-session.md](web/auth-and-session.md) |
| A **JWT**, `alg`, `kid`, weak JWT secret, token claims trusted | [identity/jwt.md](identity/jwt.md) |
| OAuth/OIDC `redirect_uri`/`state`/PKCE, account linking, issuer/audience mixup | [identity/oauth-oidc.md](identity/oauth-oidc.md) |
| SAML assertion/signature/conditions | [identity/saml.md](identity/saml.md) |
| MFA/OTP/passkey/backup-code/remember-device bypass | [identity/mfa.md](identity/mfa.md) |
| State-changing request with no anti-CSRF token | [web/csrf.md](web/csrf.md) |
| File upload, or `?file=`/path params, `../`, LFI/RFI, zip | [web/file-upload-and-path.md](web/file-upload-and-path.md) |
| A serialized blob (Java `rO0`, PHP `O:`, pickle, .NET, `ViewState`) | [web/deserialization.md](web/deserialization.md) |
| Prices/quantities/coupons/limits; "what if I do it twice / out of order" | [web/business-logic.md](web/business-logic.md) |
| Missing security headers, permissive CORS, debug on, `.git`/`.env` exposed, verbose errors | [web/misconfiguration.md](web/misconfiguration.md) |
| Front-end+back-end/CDN/proxy chain; requests "bleed" between users; poison a shared cache | [web/request-smuggling-and-desync.md](web/request-smuggling-and-desync.md) |
| `__proto__`/`constructor`/`prototype` in JSON or `a[__proto__][x]` query; JS deep-merge; Node app | [web/prototype-pollution.md](web/prototype-pollution.md) |
| API object/function auth: BOLA/BFLA, object ID swap, admin function as normal user | [api/bola-bfla.md](api/bola-bfla.md) |
| API mass assignment or full object serialization | [api/mass-assignment-data-exposure.md](api/mass-assignment-data-exposure.md) |
| GraphQL introspection/batching/depth, large limits, shadow API versions | [api/graphql-resource-limits.md](api/graphql-resource-limits.md) |
| Hard-coded key/token/password; secret in git/CI/JS bundle | [secrets-and-supply-chain/secret-detection.md](secrets-and-supply-chain/secret-detection.md) |
| A dependency — known CVE, typo/hallucinated package, lockfile | [secrets-and-supply-chain/dependency-supply-chain.md](secrets-and-supply-chain/dependency-supply-chain.md) |
| GitHub Actions / CI: `pull_request_target`, untrusted `${{ }}` in shell, mutable actions, self-hosted runners | [secrets-and-supply-chain/ci-cd-attacks.md](secrets-and-supply-chain/ci-cd-attacks.md) |
| Prompt injection, jailbreak, system-prompt leak, model output → a sink | [ai-llm/prompt-injection.md](ai-llm/prompt-injection.md) |
| Agent/tool/MCP over-permission, RAG/vector leakage, token/cost DoS | [ai-llm/agentic-and-mcp.md](ai-llm/agentic-and-mcp.md) |
| Loading a model file (`torch.load`/pickle/`.pt`/`.h5`), HF `trust_remote_code`, poisoned weights | [ai-llm/model-supply-chain.md](ai-llm/model-supply-chain.md) |
| Terraform/Dockerfile/k8s misconfig, public bucket, container as root | [cloud-and-infra/iac-and-containers.md](cloud-and-infra/iac-and-containers.md) |
| IAM wildcard/priv-esc, exposed storage, metadata creds (SSRF chain) | [cloud-and-infra/cloud-iam.md](cloud-and-infra/cloud-iam.md) |
| An `.apk`/`.ipa`; hard-coded secrets, exported component, pinning | [mobile/README.md](mobile/README.md) |
| A native binary: overflow / format string / ROP / heap (pwn) | [binary-exploitation/README.md](binary-exploitation/README.md) |
| Need to understand an unknown binary (decompile, unpack, crackme) | [reverse-engineering/README.md](reverse-engineering/README.md) |
| C/C++ / firmware / embedded **source** (secure-coding, MISRA/CERT-C, memory safety) | [secure-coding-standards.md](secure-coding-standards.md) |
| RSA/AES-CBC/XOR/hash, padding oracle, weak RNG, reused nonce | [cryptography/README.md](cryptography/README.md) |
| A PCAP / memory dump / disk image / stego / log to analyze | [forensics/README.md](forensics/README.md) |
| Need to find the attack surface first (subdomains, endpoints, OSINT) | [recon/README.md](recon/README.md) |
| Got a low/medium — *what does it unlock?* how do I escalate across a boundary? | [chaining-and-impact.md](chaining-and-impact.md) |
| Build a specific impact chain — ATO · cloud takeover · RCE · mass exfil · AI-agent | [chaining-and-impact.md](chaining-and-impact.md) → [chaining/](chaining/account-takeover.md) |
| Need to score a finding / write the report | [severity-and-triage.md](severity-and-triage.md) · [reporting-and-disclosure.md](reporting-and-disclosure.md) |
| Picking audit depth (quick/standard/deep), first-contact profiling, dependency/SCA check | [audit-modes.md](audit-modes.md) |
| Local tools may be missing; want Semgrep/gitleaks/trivy Docker fallback | [tooling/tool-preflight.md](tooling/tool-preflight.md) |
| Understand the project's *design intent* first / "is this finding deliberate?" | [design-intent.md](design-intent.md) |
| Run attacker × defender as a real diff (not a footnote); the fix→bypass loop | [red-blue.md](red-blue.md) |
| Track *coverage / recall* — every source→sink with a verdict; "what did I NOT look at?" | [audit-ledger.md](audit-ledger.md) |
| Confirm a finding by *running* a benign PoC (executed vs traced); how far the gate lets you go | [proof-and-confirmation.md](proof-and-confirmation.md) |
| Verify a *specific suspected* finding — is it real, or a false positive? | [fp-verification.md](fp-verification.md) |
| Emit a report file / scheduled or incremental (diff) re-audit using history | [continuous-audit.md](continuous-audit.md) |
| Writing the report — the *one fixed shape* every audit must emit (consistency) | [report-template.md](report-template.md) |
| Don't just fix it — *enforce* it (structural control + CI gate per category) | [enforce-forward.md](enforce-forward.md) |

Don't see your exact signal? Three fallbacks: (1) the **principle-driven** entry —
the six generative principles in [pattern-triggers.md](pattern-triggers.md) reason from *how the
system must break* rather than a symptom; (2) the target→branch table in [SKILL.md](../SKILL.md);
(3) scan the tree below and open the matching branch README (each one is itself just a routing table
to its leaves).

```
Longinus
│
├── ⛓️  SPINE (governance & lens — always applies)
│   ├── authorization-and-scope.md ....... ⛔ authorization gate, rules of engagement, legal/ethics
│   ├── design-intent.md ................ 🎯 read the design intent first → audit the gap (intent vs reality)
│   ├── red-blue.md ...................... ⚔️ bidirectional method: Blue control-map × Red sinks → the diff is the flaw
│   ├── audit-ledger.md ................. 📒 the shared surface[]/controls[] ledger — coverage/recall + the red×blue diff
│   ├── basic-vibe-triage.md ............. fast pre-leaf sweep for default creds, debug/admin surfaces, weak secrets
│   ├── pattern-triggers.md .............. 🎯 6 generative principles + ⛔ DO-NOT-report FP guards
│   ├── pattern-catalog.md ............... 🎯 code pattern → vulnerability → leaf lookup tables
│   ├── proof-and-confirmation.md ....... 🔬 run a benign PoC → Confirmed (executed) vs (traced); gate-bounded
│   ├── fp-verification.md .............. ✅ verify a suspected finding → TRUE/FALSE positive (restate→trace→gate→verdict)
│   ├── limitations.md ................... ⚠️ what this skill cannot find (honest limits)
│   ├── methodology.md ................... engagement lifecycle: recon → test → confirm → triage → report
│   ├── audit-modes.md ................... 🎚️ quick/standard/deep modes, first-contact profiling, SCA check
│   ├── chaining-and-impact.md ........... 🔗 compose findings across trust boundaries → real impact
│   │   └── chaining/ ................... deep playbooks: account-takeover · cloud-takeover · rce · data-exfiltration · ai-agent
│   │       (each: step-by-step real-world chain + 🛡️ defensive seal)
│   ├── severity-and-triage.md ........... CVSS 4.0, de-dup, false-positive control, prioritization
│   ├── enforce-forward.md ............... 🛡️ kill the bug class + gate the regression (per-category controls + templates/)
│   ├── reporting-and-disclosure.md ...... report + PoC templates, responsible disclosure
│   ├── continuous-audit.md ............. 🔁 report-file output + incremental (diff) re-audit + scheduling
│   └── report-template.md .............. 📋 the ONE fixed report shape — emit verbatim (cross-project consistency)
│
├── 🔭 recon/ ............................ ATTACK-SURFACE DISCOVERY  (dynamic targets + code inventory)
│   ├── README.md ........................ when/how to recon; passive vs active; code-as-target
│   ├── passive-recon.md ................. OSINT-safe: subdomains, certs, archives, dorking, leaks
│   ├── active-recon.md .................. probing: port/service scan, content/dir discovery, params, fuzzing
│   └── osint.md ......................... people/org/infra intel, breach data, metadata
│
├── 🌐 web/ .............................. WEB APPLICATIONS  (OWASP Top 10:2025 mapped)
│   ├── README.md ........................ web triage + Top 10:2025 → leaf map
│   ├── access-control.md ................ A01: IDOR/BOLA, priv-esc, path traversal, force-browsing, SSRF*
│   ├── injection.md ..................... A05: SQL/NoSQL/OS-command/LDAP/SSTI/XXE/header/log injection
│   ├── xss.md ........................... reflected/stored/DOM XSS, CSP bypass, mXSS
│   ├── ssrf.md .......................... SSRF (cloud metadata, internal pivot, blind/OOB)
│   ├── auth-and-session.md .............. A07: login flaws, session fixation, cookie/CSRF interplay
│   ├── csrf.md .......................... CSRF, SameSite gaps, state-changing GET
│   ├── file-upload-and-path.md .......... unrestricted upload, path traversal, LFI/RFI, archive/zip-slip
│   ├── deserialization.md ............... A08: insecure deserialization (Java/PHP/Python/.NET/Node)
│   ├── business-logic.md ................ workflow abuse, race conditions, price/quantity tampering
│   ├── misconfiguration.md .............. A02: headers, CORS, debug, defaults, verbose errors, secrets exposure
│   ├── request-smuggling-and-desync.md .. ADVANCED: HTTP smuggling/desync, HTTP2, web cache poisoning & deception
│   └── prototype-pollution.md ........... ADVANCED: client/server-side prototype pollution → XSS/RCE/SSRF
│
├── 🔌 api/ .............................. APIs  (OWASP API Security Top 10:2023)
│   ├── README.md ........................ branch router + consolidated quick scan
│   ├── bola-bfla.md ..................... object/function authorization: BOLA/BFLA
│   ├── mass-assignment-data-exposure.md . property authz: mass assignment + excessive data exposure
│   └── graphql-resource-limits.md ....... GraphQL/resource limits, inventory, unsafe upstream consumption
│
├── 🪪 identity/ ......................... AUTHN/AUTHZ PROTOCOLS
│   ├── README.md ........................ branch router + consolidated quick scan
│   ├── jwt.md ........................... JWT alg/signature/claim validation
│   ├── oauth-oidc.md .................... OAuth/OIDC redirects, state, PKCE, token mixups
│   ├── saml.md .......................... SAML assertion/signature/condition validation
│   └── mfa.md ........................... MFA/OTP/passkey/step-up enforcement
│
├── 🔑 secrets-and-supply-chain/ ......... SECRETS & DEPENDENCIES  (A03 + the #1 vibe-coding risk)
│   ├── README.md ........................ branch triage; why this is first for AI-generated code
│   ├── secret-detection.md .............. hard-coded keys/tokens, git history, env/CI leakage, rotation
│   ├── dependency-supply-chain.md ....... SCA/CVEs, slopsquatting/typosquatting, lockfiles, SBOM, SLSA
│   └── ci-cd-attacks.md ................. pwn-request, script injection, cache poisoning, mutable actions, runners
│
├── 🤖 ai-llm/ ........................... LLM / AGENT / RAG APPS  (OWASP LLM Top 10:2025)
│   ├── README.md ........................ LLM Top 10:2025 → leaf map; how to red-team an AI feature
│   ├── prompt-injection.md .............. direct/indirect injection, jailbreak taxonomy, multimodal/smuggling, output handling, exfil
│   ├── agentic-and-mcp.md ............... excessive agency, tool abuse, MCP/agent chains, RAG/vector leakage
│   └── model-supply-chain.md ........... LLM03/04: malicious model files (pickle/.pt/Keras→RCE), hub poisoning, trust_remote_code
│
├── ☁️  cloud-and-infra/ ................. CLOUD, IaC, CONTAINERS
│   ├── README.md ........................ branch triage; cloud-pentest vs config-audit
│   ├── iac-and-containers.md ............ Terraform/CFN, Dockerfile, Kubernetes, image scanning
│   └── cloud-iam.md ..................... AWS/GCP/Azure IAM, privilege escalation, exposed storage, metadata
│
├── 📱 mobile/ ........................... MOBILE APPS  (OWASP MASVS/MASTG)
│   └── README.md ........................ Android/iOS: storage, certs/pinning, IPC, hardcoded secrets, API
│
├── 💥 binary-exploitation/ .............. MEMORY CORRUPTION / PWN  (native, CTF + real)
│   └── README.md ........................ stack/heap overflows, format strings, ROP, mitigations & bypasses
│
├── 🔬 reverse-engineering/ .............. RE
│   └── README.md ........................ static/dynamic RE, disassembly/decompilation, unpacking, anti-RE
│
├── 🧷 secure-coding-standards.md ........ C/C++/embedded SOURCE compliance — CERT-C/CWE (security) · MISRA/ISO26262 (safety)
│
├── 🔐 cryptography/ ..................... CRYPTO ATTACKS  (CTF + real-world misuse)
│   └── README.md ........................ padding-oracle, ECB/CBC, weak RNG, hash/length-ext, RSA, JWT-alg, TLS
│
├── 🧪 forensics/ ........................ DFIR / FORENSICS
│   └── README.md ........................ file carving, memory (Volatility), pcap, disk, stego, log analysis
│
└── 🧰 tooling/ .......................... CURATED TOOL CATALOG
    ├── README.md ........................ best-in-class tools per branch, with safe-usage notes
    └── tool-preflight.md ................ ready/docker/missing/blocked status + Docker fallbacks
```

\* SSRF is its own leaf for depth, though OWASP Top 10:2025 folds it into A01 (Broken Access Control).

---

For target-type routing, see the fast routing table in [SKILL.md](../SKILL.md).

## Framework cross-reference (where each standard lives)

- **OWASP Top 10:2025** → `web/` (A01–A10 mapped in `web/README.md`)
- **OWASP API Security Top 10:2023** → `api/README.md`
- **OWASP LLM Top 10:2025** → `ai-llm/README.md`
- **OWASP MASVS** → `mobile/README.md`
- **OWASP WSTG / ASVS / Cheat Sheets** → cited throughout `web/`, `api/`, `identity/`
- **CWE / CAPEC / CVSS / EPSS** → `severity-and-triage.md`
- **PTES / NIST 800-115** → `methodology.md`
- **MITRE ATT&CK / ATLAS** → `cloud-and-infra/`, `ai-llm/`

All canonical source URLs are in [../RESEARCH.md](../RESEARCH.md).
