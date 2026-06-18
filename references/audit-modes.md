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
| **quick** | Fast first pass; small on-device models (≈11B); CI gate checks | **minimal detect** (stack + form factor from one listing — Step 1 + Step 2 form factor, enough to route) → the `## Mechanical scan` at the top of each relevant leaf. ONE leaf per domain (for multi-leaf domains like web, use the branch README's consolidated `## Mechanical scan`). | All "Deep analysis" sections, principle-driven reasoning, chaining-and-impact.md, CVSS scoring, **Step 0 design-intent read** — **and the standard+ disciplines: the surface/control ledger + red/blue control-map ([red-blue.md](red-blue.md)), executable PoC ([proof-and-confirmation.md](proof-and-confirmation.md))**. Use the fixed severities; results are provisional. |
| **standard** | Default. Full audit for one target. | route → leaf (full) → spine docs as needed. | Domains not relevant to this target. |
| **deep** | Comprehensive formal audit, multi-domain sweep. | Everything in standard PLUS chaining-and-impact.md, all relevant domain `Mechanical scan`s, cross-domain pivot analysis. | Nothing — full coverage. |
| **continuous / diff** | scheduled / CI re-runs on a watched project | only the **diff since the last report** (`git diff <prior-sha>..HEAD`) + a re-check of prior open findings | unchanged code — audit only what changed, append a delta ([continuous-audit.md](continuous-audit.md)) |

**Quick-mode output:** one provisional table — `File:Line | Type | Severity | Pattern | Fix`, every row
**Needs-validation**. Quick over-reports by design → **re-run `standard` on anything High/Critical before
acting** ([severity-and-triage.md](severity-and-triage.md) gates). Standard = "The loop" in
[SKILL.md](../SKILL.md). Deep = standard + [chaining-and-impact.md](chaining-and-impact.md) cross-domain
pivots (web→cloud, app→infra, model→sink) + business-impact severity overrides + the "what was NOT
tested" section ([limitations.md](limitations.md)).

---

## First-contact profiling (when you don't know where to start)

When you meet code or a target for the first time and aren't sure which leaf to open, execute these
steps **mechanically, in order**. Do not skip steps.

> **One batched recon pass feeds Steps 1–3 — don't round-trip.** The steps are listed in *reasoning*
> order, but their *scans read the same bytes*: one file listing classifies both the **stack** (Step 1)
> and the **form factor** (Step 2), and one grep set enumerates every **source→sink** for the ledger
> (Step 3). Each separate tool round-trip reprocesses the growing context. The sweep greps are
> **stack-agnostic — they do NOT depend on the listing's output**, so put the listing AND the `scan()`
> greps in **one Bash call** (don't wait to eyeball the listing before sweeping — that hesitation is the
> extra round-trip to avoid). Only the **SCA tool choice** (Step 1.5) needs the stack → run that as the
> single follow-up. Then walk Steps 0–5 classifying the output. Prefer a static-analysis oracle over the
> greps when one is installed (Step 3).
>
> ```bash
> # Detect stack + form factor — ONE listing (manifests reveal both: package.json, *.tf, Dockerfile,
> #   requirements.txt/pyproject, go.mod, Cargo.toml, pom.xml/build.gradle, Gemfile, AndroidManifest.xml…)
> find . -maxdepth 2 -not -path './.git/*' | head -100
>
> # Sweep source→sink (stack-agnostic — runs in THIS SAME call as the listing above, NOT a 2nd pass).
> # reachable: unknown on every hit.
> # scan() carries the volume guards to every grep: skip vendored/generated trees + bundles, cap line
> # width (rg honors .gitignore; these globs also catch committed node_modules/dist/*.min.js it doesn't).
> scan() { rg -n --max-columns=200 -g '!{node_modules,vendor,dist,build,target,.next,.venv}/**' -g '!*.min.js' -g '!*.lock' "$@"; }
> scan "router\.|app\.(get|post|put|delete)|@(Get|Post|Put|Delete|Patch)" .       # sources: HTTP routes
> scan "def (get|post|put|delete|patch)\b|@api_view|@app\.route" .                 # sources: handlers (\b drops get_user/getter)
> scan "\bexec\w*\s*\(|\bsystem\s*\(|\beval\s*\(|\.query\s*\(|\.raw\s*\(|innerHTML|pickle\.|torch\.load|requests\.get" .  # sinks — anchored to call-shape: skips executor/filesystem/evaluate, keeps executeQuery/execSync
> scan "middleware|guard|@auth|@login_required|verify.*token|isAuthenticated" .   # controls → Blue map (kept broad: auth recall)
> # secrets: NOT swept here — the always-first secrets branch owns the high-recall secret sweep (its 60-sec triage).
> # If a grep floods, count first (scan -c "<re>" .), then scope the path or hand off to a taint oracle.
> ```

### Step 0 — Read the design intent *(standard+; quick skips it)*

Before the greps, skim the project's own docs (`CLAUDE.md` / README / `docs/`+ADRs / `SECURITY.md` /
intent-bearing comments) and build the **Intent Brief**: purpose · *designed* trust boundaries · stated
assumptions · *documented* accepted-risks. Aim the rest of the audit at the input vectors that
**violate** it, and reconcile findings against it. Quick mode defers this — it feeds triage/severity,
which quick doesn't do. See [design-intent.md](design-intent.md).

### Step 1 — Identify the stack (from the recon listing)

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

**Form factor — what is it? (multi-select):**

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
| **none of the above** | **Generic source (fallback)** | the **universal baseline** — *never a dead end* (note below) |

**Exposure & stakes — orthogonal to form factor, sets severity (the threat-model axis):**
- **Exposure:** public-internet · internal-only · local/single-user — *who can reach it?*
- **Tenancy:** single-tenant · multi-tenant · n/a — *is cross-tenant isolation a boundary?*
- **Crown jewels:** auth · money/payments · PII · secrets · none — *what's at stake?*

These feed the Intent Brief ([design-intent.md](design-intent.md)) and **adjust severity**
([severity-and-triage.md](severity-and-triage.md)): multi-tenant → cross-tenant/BOLA is top priority +
severity↑; public-internet → exposure↑; local/single-user → most remote attacks fall away, local-privilege
+ supply-chain rise.

> **Universal baseline (the fallback) & out-of-scope.** Every audit — *any* form factor, including
> "generic source" — always runs **secrets + SCA → surface sweep → six principles**
> ([pattern-triggers.md](pattern-triggers.md)), so a novel/unclassified target is never uncovered; the
> exposure/stakes axis still sets its threat model. Targets outside the source-/app-audit model (smart
> contracts / web3, etc.): **say so and point to a specialized tool** — flag the coverage boundary, never
> silently mis-route.

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

**Fallback — grep enumeration:** the `scan()` greps in the **one batched recon pass** at the top of this
section already enumerate sources / sinks / controls in a single round-trip, anchored to call-shape and
scoped past vendored/generated trees (don't re-run them here — each round-trip reprocesses the growing
context). Turn each hit into a `surface` row, **`reachable: unknown` on every hit** until a taint oracle
or a read proves the path. (Secrets are swept by the always-first secrets branch, not here.)

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
