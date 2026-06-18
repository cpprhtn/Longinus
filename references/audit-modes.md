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
| **quick** | Fast first pass; small on-device models (≈11B); CI gate checks | route → the `## Mechanical scan` section at the top of each relevant leaf. ONE leaf per domain (for multi-leaf domains like web, use the branch README's consolidated `## Mechanical scan`). | All "Deep analysis" sections, principle-driven reasoning, chaining-and-impact.md, CVSS scoring — **and the standard+ disciplines: the surface/control ledger + red/blue control-map ([red-blue.md](red-blue.md)), executable PoC ([proof-and-confirmation.md](proof-and-confirmation.md))**. Use the fixed severities; results are provisional. |
| **standard** | Default. Full audit for one target. | route → leaf (full) → spine docs as needed. | Domains not relevant to this target. |
| **deep** | Comprehensive formal audit, multi-domain sweep. | Everything in standard PLUS chaining-and-impact.md, all relevant domain `Mechanical scan`s, cross-domain pivot analysis. | Nothing — full coverage. |
| **continuous / diff** | scheduled / CI re-runs on a watched project | only the **diff since the last report** (`git diff <prior-sha>..HEAD`) + a re-check of prior open findings | unchanged code — audit only what changed, append a delta ([continuous-audit.md](continuous-audit.md)) |

### Quick mode instructions

1. Pass the [authorization gate](authorization-and-scope.md).
2. Identify the stack (Step 1 below).
3. Run the dependency health check (Step 1.5 below).
4. Route to the matching domain leaf (or the branch README's `## Mechanical scan` for multi-leaf domains).
5. In each leaf/branch, read ONLY the `## Mechanical scan` section.
6. Execute each numbered STEP (grep), apply the SKIP conditions, and report any remaining match using
   the fixed severity and the leaf's quick-mode output template.
7. Do NOT perform CVSS scoring, chaining analysis, principle-driven reasoning, the red/blue control-map,
   or executable PoCs — those are **standard+** disciplines.
8. Report as a single table: `File:Line | Type | Severity (fixed) | Pattern matched | Fix`. This **provisional
   table is quick mode's deliverable** — not the full [report-template.md](report-template.md) (that is the
   standard/deep output). Every quick-mode row is effectively **Needs-validation**; a `standard` re-run is
   what confirms it (executed/traced) and assigns a real severity.

> ⚠️ **Quick-mode severities are provisional.** The fixed per-pattern severities skip the reachability,
> CVSS, and chaining checks that prevent inflation ([severity-and-triage.md](severity-and-triage.md)
> "Mandatory severity gates"). Quick mode trades precision for speed and **may over-report** — treat its
> output as a triage signal, and **re-run `standard` mode on anything flagged High/Critical before
> acting on it.**

### Standard mode instructions

Default. Profile → **surface sweep (build the ledger)** → route → leaf (full) → PoC → triage → report.
See "The loop" in [SKILL.md](../SKILL.md) and the surface sweep below.

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

### Step 0 — Read the design intent

Before the greps, skim the project's own docs (`CLAUDE.md` / README / `docs/`+ADRs / `SECURITY.md` /
intent-bearing comments) and build the **Intent Brief**: purpose · *designed* trust boundaries · stated
assumptions · *documented* accepted-risks. Aim the rest of the audit at the input vectors that
**violate** it, and reconcile findings against it. See [design-intent.md](design-intent.md).

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

### Step 2 — Profile the project (form factor × exposure × what's at stake)

Build a short **project profile** — orthogonal axes that route the Red domains *and* set the severity
floor. A few lines; it runs every audit.

**a) Form factor — what is it? (multi-select):**

| Signal | Form factor | Red domain |
|---|---|---|
| HTTP routes/controllers/templates | **Web app** | `web/` |
| GraphQL/REST endpoints, no UI | **API** | `api/` |
| `openai`/`anthropic`/`langchain`/model loading | **AI/LLM / agent** | `ai-llm/` |
| `*.tf` / k8s `*.yaml` / `Dockerfile` | **Cloud / IaC** | `cloud-and-infra/` |
| `AndroidManifest.xml`/`*.xcodeproj`/`*.swift`/`*.kt` | **Mobile** | `mobile/` |
| compiled binary / firmware | **Native binary** | `reverse-engineering/` → `binary-exploitation/` |
| `argparse`/`click`/`cobra`/`commander`, no server | **CLI / script** | secrets + input handling (argv/env, local privilege, supply chain) |
| published package, no entrypoint (others import it) | **Library / SDK** | "what can a hostile caller / input do" → input→sink + deserialization |
| Electron/Tauri, or webextension `manifest.json` | **Desktop / browser-extension** | `web/` sinks + local IPC/filesystem + update channel |
| ETL/Spark/Airflow/notebook/training script | **Data / ML pipeline** | deserialization (pickle/`torch.load`) + data poisoning + secrets in notebooks |
| Lambda/Cloud-Function/webhook handler | **Serverless / webhook** | `api/` + cloud IAM (the trigger input + the function's role) |
| **none of the above** | **Generic source (fallback)** | the **universal baseline** (c) — *never a dead end* |

**b) Exposure & stakes — orthogonal to form factor, sets severity (the threat-model axis):**
- **Exposure:** public-internet · internal-only · local/single-user — *who can reach it?*
- **Tenancy:** single-tenant · multi-tenant · n/a — *is cross-tenant isolation a boundary?*
- **Crown jewels:** auth · money/payments · PII · secrets · none — *what's at stake?*

These feed the Intent Brief ([design-intent.md](design-intent.md)) and **adjust severity**
([severity-and-triage.md](severity-and-triage.md)): multi-tenant → cross-tenant/BOLA is top priority +
severity↑; public-internet → exposure↑; local/single-user → most remote attacks fall away, local-privilege
+ supply-chain rise.

**c) Universal baseline — the fallback that guarantees coverage.** *Whatever* the form factor — including
"generic source" — every audit always runs **secrets + dependency/SCA** (first), the **surface sweep**
(every source→sink, Step 3), and the **six generative principles** ([pattern-triggers.md](pattern-triggers.md),
which derive vulnerability classes for code with no named playbook). So an unclassified or novel target is
never uncovered — it falls through to this baseline, and the exposure/stakes axis still sets its threat model.

**d) Recognized but out of operating model.** Deliberately out-of-scope targets (smart contracts / web3,
and anything outside the source-/app-audit operating model): **say so and point to a specialized tool** —
flag it as a coverage boundary, never silently mis-route it.

### Step 3 — Surface sweep: build the ledger (the recall oracle)

This step enumerates **every source→sink** into the `surface[]` table of the
[Audit Ledger](audit-ledger.md) — so coverage is *measured* and un-examined surface is *visible*, not
silent. **Prefer a static-analysis oracle** as the recall + reachability engine; fall back to greps when
none is installed (graceful degradation — like the SCA step above).

**Preferred — static-analysis oracle (sound taint = high recall + a reachability signal):**

| Engine | Use for | Note |
|---|---|---|
| [Semgrep](https://semgrep.dev/) (`semgrep --config auto`, taint rules) | multi-language source→sink taint | fast, OSS rulesets per language |
| [CodeQL](https://codeql.github.com/) | deep taint / data-flow on big codebases | the strongest reachability oracle |
| [Joern](https://joern.io/) | code-property-graph queries, C/C++/JVM/JS | when you need custom flow queries |
| language LSP / type info | call-graph for "is this reached" | complements the above |

The oracle does what the LLM is bad at — **deterministic enumeration and reachability** — and the LLM
does what *it* is good at on top: semantic triage, chaining, business impact, intent reconciliation.
Each taint result becomes a `surface` row (`reachable: true|false`); each unreached candidate is
`reachable: unknown` → Needs-validation, never a confident finding.

**Fallback — grep enumeration (set `reachable: unknown` on every hit):**

```bash
# Sources — where untrusted input enters
rg -n "router\.|app\.(get|post|put|delete)|@(Get|Post|Put|Delete|Patch)" .
rg -n "def (get|post|put|delete|patch)|@api_view|@app\.route" .

# Sinks — where input becomes dangerous
rg -n "execute|\.query|\.raw\(|exec|system|eval|innerHTML|pickle|torch\.load|requests\.get" .

# Auth patterns (or lack thereof) — feeds Blue's controls[] map
rg -n "middleware|guard|@auth|@login_required|verify.*token|isAuthenticated" .

# Secrets (immediate high-confidence findings)
rg -n "(api[_-]?key|secret|password|token)\s*[:=]" -i .
```

> **Coverage = `examined / total` sinks.** Record it in the report header ([report-template.md](report-template.md))
> and name the `not-examined` rows in §7. Recall (did you look?) is the orthogonal twin of precision (are
> you sure?) — see [audit-ledger.md](audit-ledger.md). The `auth pattern` hits seed **Blue's**
> expected-control map for the [red×blue diff](red-blue.md).

### Step 4 — Route to the correct leaf

Jump to the matching row in the fast routing table in [SKILL.md](../SKILL.md). If multiple domains
apply (a web app that also uses an LLM), prioritize:
1. `secrets-and-supply-chain/` (always first — `## Mechanical scan` / 60-second triage)
2. Primary domain (web / api / ai-llm / cloud / mobile)
3. Secondary domains

### Step 5 — For source-code audits, also use the pattern catalog

Open [pattern-catalog.md](pattern-catalog.md) — it maps code patterns directly to vulnerability
checks. Scan for the patterns; each match tells you exactly which leaf to open and what to confirm.

---

## Domain quick checks

Every domain README has a **`## Mechanical scan`** section (grep steps + fixed severities + a quick-mode
output template). After routing to a domain, run its `Mechanical scan` before going deeper into leaves.

Back to [SKILL.md](../SKILL.md) · [tree](00-map.md).
