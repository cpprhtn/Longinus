# 🎚️ Audit modes, first-contact profiling & dependency (SCA) check

Operational detail for *running* an audit. [SKILL.md](../SKILL.md) selects the mode and routes; this
file holds the step-by-step procedure, the first-contact profiling sequence, and the dependency
health check. Pull it in once you've passed the [authorization gate](authorization-and-scope.md) and
know you're starting an audit.

---

## Three modes (depth & token budget)

Default is **standard** unless the user specifies otherwise ("run a quick security audit" → quick;
"deep audit this repo" → deep).

| Mode | When to use | What to read | What to skip |
|---|---|---|---|
| **quick** | Fast first pass; small on-device models (≈11B); CI gate checks | route → the `## Mechanical scan` section at the top of each relevant leaf. ONE leaf per domain (for multi-leaf domains like web, use the branch README's consolidated `## Mechanical scan`). | All "Deep analysis" sections, principle-driven reasoning, chaining-and-impact.md, CVSS scoring. Use the fixed severities from the mechanical scan. |
| **standard** | Default. Full audit for one target. | route → leaf (full) → spine docs as needed. | Domains not relevant to this target. |
| **deep** | Comprehensive formal audit, multi-domain sweep. | Everything in standard PLUS chaining-and-impact.md, all relevant domain `Mechanical scan`s, cross-domain pivot analysis. | Nothing — full coverage. |

### Quick mode instructions

1. Pass the [authorization gate](authorization-and-scope.md).
2. Identify the stack (Step 1 below).
3. Run the dependency health check (Step 1.5 below).
4. Route to the matching domain leaf (or the branch README's `## Mechanical scan` for multi-leaf domains).
5. In each leaf/branch, read ONLY the `## Mechanical scan` section.
6. Execute each numbered STEP (grep), apply the SKIP conditions, and report any remaining match using
   the fixed severity and the leaf's quick-mode output template.
7. Do NOT perform CVSS scoring, chaining analysis, or principle-driven reasoning.
8. Report as a single table: `File:Line | Type | Severity (fixed) | Pattern matched | Fix`.

> ⚠️ **Quick-mode severities are provisional.** The fixed per-pattern severities skip the reachability,
> CVSS, and chaining checks that prevent inflation ([severity-and-triage.md](severity-and-triage.md)
> "Mandatory severity gates"). Quick mode trades precision for speed and **may over-report** — treat its
> output as a triage signal, and **re-run `standard` mode on anything flagged High/Critical before
> acting on it.**

### Standard mode instructions

Default. Profile → route → leaf (full) → PoC → triage → report. See "The loop" in [SKILL.md](../SKILL.md).

### Deep mode instructions

1. Run standard mode for the primary domain.
2. Sweep ALL secondary domains (run each domain's `## Mechanical scan`).
3. Open [chaining-and-impact.md](chaining-and-impact.md) and systematically test cross-domain pivots
   (web→cloud, app→infra, model→sink).
4. Run the full severity gates with business-impact overrides ([severity-and-triage.md](severity-and-triage.md)).
5. Produce a comprehensive report per [reporting-and-disclosure.md](reporting-and-disclosure.md),
   including the "what was NOT tested" section from [limitations.md](limitations.md).

---

## First-contact profiling (when you don't know where to start)

When you meet code or a target for the first time and aren't sure which leaf to open, execute these
steps **mechanically, in order**. Do not skip steps.

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

### Step 1.5 — Dependency health check / SCA (after stack, before form factor)

Run the appropriate SCA tool for the stack from Step 1:

| Stack | Command |
|---|---|
| Node.js | `npm audit --omit=dev` |
| Python | `pip-audit` or `osv-scanner -r .` |
| Go | `govulncheck ./...` |
| Rust | `cargo audit` |
| Ruby | `bundler-audit` |
| Java / Kotlin | `mvn dependency-check:check` or `trivy fs .` |
| Multi-ecosystem | `trivy fs .` or `osv-scanner -r .` |

> **If no SCA tool is installed** (common in a fresh audit env): do **not** guess CVEs from memory.
> Instead — extract the dependency names + pinned versions from the lockfile/manifest and check them
> against the **[OSV](https://osv.dev/) / [GitHub Advisory](https://github.com/advisories)** database
> (by name+version), or simply report **"SCA not run — scanner unavailable"** in the Dependency Health
> section. A scanner you couldn't run is a *coverage gap to disclose*, never a reason to invent findings.

**Severity rule (mandatory — aligns with the severity gates):**
- CVE with a **confirmed reachable code path** → severity per normal triage.
- CVE with **no confirmed reachability** → **Info / "patch anyway"** only; do NOT rate High/Critical
  ([severity-and-triage.md](severity-and-triage.md)).
- **Hallucinated CVE** (cannot cite it with a real link) → **DO NOT report** ([limitations.md](limitations.md)).

Report dependency findings in a separate **"Dependency Health"** section, distinct from
application-level findings. Full SCA playbook:
[secrets-and-supply-chain/dependency-supply-chain.md](secrets-and-supply-chain/dependency-supply-chain.md).

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

Jump to the matching row in the fast routing table in [SKILL.md](../SKILL.md). If multiple domains
apply (a web app that also uses an LLM), prioritize:
1. `secrets-and-supply-chain/` (always first — `## Mechanical scan` / 60-second triage)
2. Primary domain (web / api / ai-llm / cloud / mobile)
3. Secondary domains

### Step 5 — For source-code audits, also use the pattern trigger table

Open [pattern-triggers.md](pattern-triggers.md) — it maps code patterns directly to vulnerability
checks. Scan for the patterns; each match tells you exactly which leaf to open and what to confirm.

---

## Domain quick checks

Every domain README has a **`## Mechanical scan`** section (grep steps + fixed severities + a quick-mode
output template). After routing to a domain, run its `Mechanical scan` before going deeper into leaves.

Back to [SKILL.md](../SKILL.md) · [tree](00-map.md).
