---
name: longinus
description: This skill should be used when the user asks to "run a security audit", "find vulnerabilities", "pentest this", "do a bug bounty", "secure my app", "review my code for security", "check for CVEs/secrets/injection", "red team my LLM/agent", or mentions "Longinus / 롱기누스 / 보안 점검". Longinus is a security testing & detection skill: it profiles a target (source repo, web app, API, LLM/agent app, mobile app, binary, or cloud config), gates on authorization, jumps to the relevant offensive playbook in a domain tree, probes for weaknesses, then reports triaged findings with proof-of-concept and concrete fixes.
version: 0.2.0
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

## 🎚️ Audit mode (select before routing)

Three modes control depth and token budget. Default is **standard** unless the user
specifies otherwise (e.g., "run a quick security audit" or "deep audit this repo").

| Mode | When to use | What to read | What to skip |
|---|---|---|---|
| **quick** | Fast first pass; small on-device models (11B); CI gate checks | SKILL.md (this file) → route → `## Mechanical scan` section at the top of each relevant leaf. ONE leaf per domain. | All "Deep analysis" sections, principle-driven reasoning, chaining-and-impact.md, CVSS scoring. Use fixed severities from the mechanical scan. |
| **standard** | Default. Full audit for one target. | SKILL.md → route → leaf (full) → spine docs as needed. | Domains not relevant to this target. |
| **deep** | Comprehensive formal audit, multi-domain sweep. | Everything in standard PLUS: chaining-and-impact.md, all relevant domain quick-checks, cross-domain pivot analysis. | Nothing — full coverage. |

### Quick mode instructions

In quick mode, follow this exact procedure:
1. Pass the authorization gate (above).
2. Identify the stack (Step 1 below).
3. Run the dependency health check (Step 1.5 below).
4. Route to the matching domain leaf.
5. In each leaf, read ONLY the `## Mechanical scan` section at the top.
6. Execute each numbered STEP (grep command), apply the SKIP conditions, report
   any remaining matches as findings using the fixed severity and the fill-in
   output template.
7. Do NOT perform CVSS scoring, chaining analysis, or principle-driven reasoning.
8. Report using a simplified format: one table of findings with columns
   `[File:Line | Type | Severity (fixed) | Pattern matched | Fix]`.

### Standard mode instructions

Default behavior. Profile → route → leaf (full) → PoC → triage → report.
See "The loop" section below.

### Deep mode instructions

1. Run standard mode for the primary domain.
2. Then sweep ALL secondary domains (run each domain's quick checks from its README).
3. Open [chaining-and-impact.md](references/chaining-and-impact.md) and systematically
   test cross-domain pivots (e.g., web→cloud, app→infra, model→sink).
4. Run the full severity gates with business-impact overrides.
5. Produce a comprehensive report per
   [reporting-and-disclosure.md](references/reporting-and-disclosure.md).

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
   [pattern-triggers.md](references/pattern-triggers.md) tell you *where it must break*, complementing
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

### Step 1.5 — Dependency health check (after stack, before form factor)

Run the appropriate SCA tool for the stack identified in Step 1:

| Stack | Command |
|---|---|
| Node.js | `npm audit --omit=dev` |
| Python | `pip-audit` or `osv-scanner -r .` |
| Go | `govulncheck ./...` |
| Rust | `cargo audit` |
| Ruby | `bundler-audit` |
| Java / Kotlin | `mvn dependency-check:check` or `trivy fs .` |
| Multi-ecosystem | `trivy fs .` or `osv-scanner -r .` |

**Severity rule (mandatory — aligns with the severity gates):**
- CVE with **confirmed reachable code path** in the app → severity per normal triage
- CVE with **no confirmed reachability** → **Info / "patch anyway"** only; do NOT rate
  as High/Critical (see [severity-and-triage.md](references/severity-and-triage.md))
- Hallucinated CVE (cannot verify it exists) → **DO NOT report**

Report dependency findings in a separate **"Dependency Health"** section of the report,
distinct from application-level findings. Full SCA playbook:
[secrets-and-supply-chain/dependency-supply-chain.md](references/secrets-and-supply-chain/dependency-supply-chain.md).

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

## ⏱️ Domain quick checks

Each domain README has a **"first 5 minutes"** checklist. After routing to a domain,
run its quick checks before going deeper into leaves.

---

## Operating principles

- **Prove it or park it** — reproducible PoC or it's unconfirmed. *This is the crown jewel:* an LLM's
  failure mode is confident hallucinated bugs, so low false positives are what earn trust.
- **Think in principles, report in chains** — derive bugs from the six generative principles
  ([pattern-triggers.md](references/pattern-triggers.md)); rate them by *chained* impact
  ([chaining-and-impact.md](references/chaining-and-impact.md)), never in isolation.
- **Fix-forward** — every finding ends in a remediation a dev can apply today.
- **Stay in scope, stay non-destructive** — when in doubt, narrow.
- **Read minimally** — one leaf at a time; depth only on demand.
- **Teach while you test** — briefly say *why* it's exploitable.
- **One audit, one session** — repeated audits in a single conversation contaminate context:
  findings from project A bias the analysis of project B (confirmation bias). Run `/clear`
  between audits of different targets. See [limitations.md](references/limitations.md).

The full navigable tree (and the signal→file jump table) is in
[references/00-map.md](references/00-map.md).
