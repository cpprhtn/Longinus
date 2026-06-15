---
name: longinus
description: This skill should be used when the user asks to "run a security audit", "find vulnerabilities", "pentest this", "do a bug bounty", "secure my app", "review my code for security", "check for CVEs/secrets/injection", "red team my LLM/agent", or mentions "Longinus / 롱기누스 / 보안 점검". Longinus is a security testing & detection skill: it profiles a target (source repo, web app, API, LLM/agent app, mobile app, binary, or cloud config), gates on authorization, jumps to the relevant offensive playbook in a domain tree, probes for weaknesses, then reports triaged findings with proof-of-concept and concrete fixes.
version: 0.1.0
---

# Longinus — Offensive Security for Defenders

Longinus is "the spear that pierces your own armor first." You play attacker so the developer finds
and fixes the hole before a real adversary does. Output is never "you got popped" — it is **a ranked
list of findings, each with a reproducible proof-of-concept and a concrete fix.**

Built for three realities: (1) vibe-coded apps ship fast and insecure (~45% of AI-generated code has a
known weakness, ~2.7× the density of hand-written code); (2) bug bounty / pentest is structured offense
on an *authorized* target; (3) CTF specialists go **deep, not wide** — pick a sub-field and descend.

---

## 🌳 How to use this skill — DON'T read the whole tree

**This is a tree you traverse, not a manual you read front-to-back.** Loading everything is slow and
pollutes your focus — exactly wrong when you're racing to find a hole. The rule:

1. **Pass the gate** (below) — always; it's the only mandatory read.
2. **Identify the target / the signal** you're looking at.
3. **Jump to ONE leaf** via the routing table below (or the finer *signal → file* table in
   [references/00-map.md](references/00-map.md)). Open *only* that file.
4. **Go deeper only when that leaf tells you to** (leaves link to siblings/`../` when a real chain
   appears). Each leaf is self-contained — one bug class, find→confirm→fix. Read it, act, move on.
5. Pull in the **spine docs only when that phase arrives** (rate a finding → `severity-and-triage.md`;
   write it up → `reporting-and-disclosure.md`; need the full lifecycle → `methodology.md`).

> Minimize reads. A typical path is **gate → one leaf → fix**, or for a fuzzy target
> **gate → 00-map jump table → one leaf**. If you've opened five docs without testing anything, stop
> and jump straight to the leaf that matches what you actually see.

---

## ⛔ Step 0 — Authorization gate (mandatory, every time)

**Before any active probing, confirm you're allowed to test the target.** This is what separates
security work from a crime.

| Target | Mode | Authorization |
|---|---|---|
| Source code / repo the user controls | **Static audit** (read-only) | Implicit — proceed |
| User's own app on localhost / their staging | Static + light dynamic | Implicit; confirm it's theirs & non-prod |
| Third-party live site / API / IP | **Dynamic** | **Explicit** — bug-bounty scope, signed engagement, or written permission |
| CTF / deliberately-vulnerable lab | Dynamic | Implicit — that's the point |

No authorization for a third-party target → **stop**; offer the static/code-audit path on owned code, or
lab/CTF practice. Always refuse to weaponize for malicious ends (mass-targeting, DoS on live third
parties, malware, evasion on systems the user doesn't own). Default to **read-only / non-destructive**:
prove a bug with a canary/benign OOB callback, don't exfiltrate or pivot. Full rules of engagement:
[references/authorization-and-scope.md](references/authorization-and-scope.md).

---

## 🧭 Fast routing — target → where to start (one hop)

Pick the row that matches the target and open that file. Don't open the others.

| The target is… | Jump straight to |
|---|---|
| **My own repo (pre-launch audit)** | [secrets-and-supply-chain/README.md](references/secrets-and-supply-chain/README.md) → run the 60-sec triage, then the rows below for what the app does |
| **A web app** | [web/README.md](references/web/README.md) (Top-10:2025 → leaf map) — or skip straight to a leaf via the [00-map jump table](references/00-map.md) |
| **An API (REST/GraphQL)** | [api/README.md](references/api/README.md) |
| **Login / OAuth / JWT / SSO** | [identity/README.md](references/identity/README.md) |
| **An LLM / agent / RAG app** | [ai-llm/README.md](references/ai-llm/README.md) |
| **Cloud account / Terraform / k8s** | [cloud-and-infra/README.md](references/cloud-and-infra/README.md) |
| **A mobile app** | [mobile/README.md](references/mobile/README.md) |
| **A native binary / firmware** | [reverse-engineering/](references/reverse-engineering/README.md) → [binary-exploitation/](references/binary-exploitation/README.md) |
| **A CTF challenge** | the matching specialist branch (crypto / forensics / pwn / reverse / web) |
| **A live target needing recon first** | [recon/README.md](references/recon/README.md) |

Already know the *symptom* (a URL the server fetches, an `id` in the path, a serialized cookie…)? Skip
this table — go to the **signal → exact-file jump table at the top of
[references/00-map.md](references/00-map.md)** and open the single matching leaf.

---

## The loop (expand a phase only when you reach it)

1. **Profile** the target — form factor, stack, trust boundaries (where untrusted input enters), crown
   jewels. For a repo, build the route/sink inventory ([recon/README.md](references/recon/README.md)
   has the greps). Bring the **attacker's lens** here — the six generative principles in
   [attacker-mindset.md](references/attacker-mindset.md) tell you *where it must break*, complementing
   the symptom-driven jump table.
2. **Recon** (dynamic targets) — enumerate surface: [recon/](references/recon/README.md).
3. **Test** — open the matching leaf, run its find→confirm steps. Default non-destructive.
4. **Confirm** — reproducible PoC or it goes in the "needs validation" bucket. *Prove it or park it.*
5. **Chain** — escalate every low/medium across a trust boundary; the *chained* impact is what you
   report. Patterns + discipline: [chaining-and-impact.md](references/chaining-and-impact.md).
6. **Triage & report** — [severity-and-triage.md](references/severity-and-triage.md) →
   [reporting-and-disclosure.md](references/reporting-and-disclosure.md).

Full detail (only if you need it): [references/methodology.md](references/methodology.md).

---

## 🔬 First-contact profiling (when you don't know where to start)

When you encounter code or a target for the first time and aren't sure which leaf to
open, execute these steps **mechanically, in order**. Do not skip steps.

### Step 1 — Identify the stack (look for these files)

| File / pattern | Stack |
|---|---|
| `package.json` / `node_modules` | Node.js / JavaScript/TypeScript |
| `requirements.txt` / `pyproject.toml` / `Pipfile` / `setup.py` | Python |
| `go.mod` / `go.sum` | Go |
| `Cargo.toml` | Rust |
| `pom.xml` / `build.gradle` | Java / Kotlin |
| `Gemfile` / `config/routes.rb` | Ruby / Rails |
| `Dockerfile` / `docker-compose.yml` | Containerized |
| `*.tf` / `terraform/` | IaC (Terraform) |
| `.env` / `config/` with secrets | **Check for secrets immediately** |

### Step 2 — Identify the form factor (what does it do?)

| Signal | Form factor | Jump to |
|---|---|---|
| HTTP routes/controllers/handlers | **Web app** | `web/` branch |
| GraphQL schema / REST endpoints without UI | **API** | `api/` branch |
| `openai` / `anthropic` / `langchain` / model loading code | **AI/LLM app** | `ai-llm/` branch |
| `*.tf` / `*.yaml` (k8s) / `Dockerfile` only | **Cloud/IaC** | `cloud-and-infra/` branch |
| `AndroidManifest.xml` / `*.xcodeproj` / `*.swift` / `*.kt` | **Mobile** | `mobile/` branch |

### Step 3 — Map entry points (run these greps, adapt to language)

```bash
# Routes / endpoints (where untrusted input enters)
rg -n "router\.|app\.(get|post|put|delete)|@(Get|Post|Put|Delete|Patch)" .
rg -n "def (get|post|put|delete|patch)|@api_view|@app\.route" .

# Sinks (where input becomes dangerous)
rg -n "execute|\.query|\.raw\(|exec|system|eval|innerHTML|pickle|torch\.load|requests\.get" .

# Auth patterns (or lack thereof)
rg -n "middleware|guard|@auth|@login_required|verify.*token|isAuthenticated" .

# Secrets (immediate high-confidence findings)
rg -n "(api[_-]?key|secret|password|token)\s*[:=]" -i .
```

### Step 4 — Route to the correct leaf

Based on steps 1–3, jump to the matching row in the fast routing table above. If multiple
domains apply (e.g., a web app that also uses an LLM), prioritize in this order:
1. `secrets-and-supply-chain/` (always first — 60-second triage)
2. Primary domain (web / api / ai-llm / cloud / mobile)
3. Secondary domains

### Step 5 — For source-code audits, also use the pattern trigger table

Open [references/pattern-triggers.md](references/pattern-triggers.md) — it maps code
patterns directly to vulnerability checks. Scan for the patterns; each match tells you
exactly which leaf to open and what to confirm.

---

## ⏱️ Domain-specific "first 5 minutes" quick checks

### AI / LLM app — first 5 checks
1. Find all model loading calls → `trust_remote_code=True`? pickle-based? unpinned model refs?
2. Enumerate all tool/function definitions → what real permissions does each have?
3. Trace where model output goes → does it reach `innerHTML` / SQL / shell / `eval` / URL fetch?
4. Find RAG/vector queries → is there a tenant/user filter at query time?
5. Read the system prompt → does it contain secrets or sensitive logic?

### Cloud / IaC — first 5 checks
1. `rg "Action.*\\*|Resource.*\\*|public-read|allUsers|0\\.0\\.0\\.0/0"` → immediate red flags
2. Check for IMDSv1 (`http_tokens = "optional"` or missing metadata block)
3. Check Dockerfiles for `USER root` / unpinned base images / secrets in build args
4. Check for secrets in `.tf` / `tfvars` / `docker-compose.yml` / env files
5. Check k8s manifests for `privileged: true` / `hostPath` / `cluster-admin` binding

### API-only — first 5 checks
1. Build endpoint inventory from route definitions
2. For each endpoint with an object ID: is ownership checked? → BOLA sweep
3. Are admin endpoints guarded by role/permission middleware? → BFLA sweep
4. Check response serialization: do endpoints return full model objects? → data exposure
5. Is there rate limiting on auth endpoints (login, OTP, reset)?

### Mobile app — first 5 checks
1. Search for hardcoded keys/secrets in source / `strings` output
2. Check `AndroidManifest.xml` for exported Activities/Services/Receivers
3. Look for certificate pinning (or lack thereof) in network config
4. Check local storage for sensitive data stored unencrypted (SharedPreferences, Keychain misuse)
5. Identify the API backend → apply the API quick checks above

---

## Operating principles

- **Prove it or park it** — reproducible PoC or it's unconfirmed. *This is the crown jewel:* an LLM's
  failure mode is confident hallucinated bugs, so low false positives are what earn trust.
- **Think in principles, report in chains** — derive bugs from the six generative lenses
  ([attacker-mindset.md](references/attacker-mindset.md)); rate them by *chained* impact
  ([chaining-and-impact.md](references/chaining-and-impact.md)), never in isolation.
- **Fix-forward** — every finding ends in a remediation a dev can apply today.
- **Stay in scope, stay non-destructive** — when in doubt, narrow.
- **Read minimally** — one leaf at a time; depth only on demand.
- **Teach while you test** — briefly say *why* it's exploitable.

The full navigable tree (and the signal→file jump table) is in
[references/00-map.md](references/00-map.md).
