# 🛡️ Enforce-forward — kill the bug class, gate the regression

> **This is the *defender lens* of a two-lens method.** Offense drives — you reach this doc *because* an
> attack ([pattern-triggers.md](pattern-triggers.md)) reached a sink and a control was missing or
> bypassable ([chaining-and-impact.md](chaining-and-impact.md)). Enforcement is how you *seal* what the
> attack exposed; it is not a checklist run on its own.
>
> **Two entry points, same matrix:** (1) *after* a finding — the gate that seals it (the primary use); and
> (2) *up front* — **Blue** consults the per-category matrix below as a **catalog of expected controls** to
> build the control-map it diffs Red against ([red-blue.md](red-blue.md)). That up-front read is a
> *reference*, still offense-led (the finding is the diff) — not this doc run as a standalone checklist.

A finding does **not** end at a one-line patch. It ends at **(1)** the *structural control* that makes
the whole class impossible, and **(2)** the *enforcement gate* that keeps it that way. The deliverable
shifts from *"found N instances of X"* to *"the codebase lacks control C; here is the layer that adds
it and the CI gate that fails the build without it."* That is the scanner-vs-engineer line on the
defensive side — and the highest-value output for the **secure-my-app** audience.

## The law (per category)

> **Detect the category → anchor to that category's canonical standard → enforce the structural control
> with a gate.**

MISRA is *not* a universal rule — it's automotive/embedded's *instance* of "the canonical standard."
The transferable move is the **enforcement layer**: every category has (a) a gold-standard, (b) a
structural control that kills its worst class, and (c) a gate that mandates the control.

## Three levels of fix (always push down this list)

| Level | What | Problem |
|---|---|---|
| 1. **Patch the instance** | fix this one line | regresses — the next dev re-introduces it |
| 2. **Structural control** | a *central layer* kills the class (output encoder, parameterized query, a validation schema, authz middleware, IMDSv2, a keystore) | needs to be *kept* present |
| 3. **Enforced gate** | CI / lint / policy-as-code / pre-commit **fails the build** if the control is absent | ✅ class stays dead, regression-proof |

Report level 1 as the immediate fix, but **always recommend levels 2–3** as the real remediation.

## Per-category enforcement matrix

For each category: the canonical **standard**, the **structural control** that kills the worst class,
and the **gate** (with a runnable template) that enforces it.

| Category | Canonical standard | Structural control (kills the class) | Enforced gate → template |
|---|---|---|---|
| **Web** ([web/](web/README.md)) | OWASP Top 10:2025 + **ASVS** (verification) | central **output-encoding** + parameterized queries + an **input-validation schema** + CSP | Semgrep/ESLint-security in CI + a validation layer → [templates/validation-layer.md](templates/validation-layer.md) · [templates/ci-gates.md](templates/ci-gates.md) |
| **API** ([api/](api/README.md)) | OWASP API Top 10 + **ASVS** | **schema validation** (OpenAPI request validation) + per-object authz + rate limit | request-validation middleware + contract test → [templates/validation-layer.md](templates/validation-layer.md) |
| **Identity** ([identity/](identity/README.md)) | **NIST 800-63** + OAuth/OIDC RFCs + **FAPI** | library token-verify (pinned alg + audience + exp) + mandatory `state`/PKCE | a JWT-verify config lint/test gate → [templates/validation-layer.md](templates/validation-layer.md) |
| **Mobile** ([mobile/](mobile/README.md)) | OWASP **MASVS/MASTG** | platform **keystore** + cert pinning + no client-side trust | MobSF/lint in CI; pinning config asserted in build |
| **Cloud / IaC** ([cloud-and-infra/](cloud-and-infra/README.md)) | **CIS Benchmarks** + well-architected | least-privilege IAM + **IMDSv2** + private-by-default | **checkov/tfsec + OPA/conftest** policy gate → [templates/policy-as-code.md](templates/policy-as-code.md) |
| **AI / LLM** ([ai-llm/](ai-llm/README.md)) | OWASP LLM Top 10 + **NIST AI RMF** + ATLAS | **output validation** + tool-arg allowlist + tenant isolation + human-in-the-loop | output-validation layer + **model-scan** (modelscan) gate → [templates/validation-layer.md](templates/validation-layer.md) · [templates/ci-gates.md](templates/ci-gates.md) |
| **Secrets / Supply chain** ([secrets-and-supply-chain/](secrets-and-supply-chain/README.md)) | **SLSA** + NIST **SSDF** + OpenSSF Scorecard | no secrets in code + pinned deps + build provenance | **gitleaks pre-commit + SCA gate + lockfile pin** → [templates/pre-commit-and-secrets.md](templates/pre-commit-and-secrets.md) · [templates/ci-gates.md](templates/ci-gates.md) |
| **CI/CD** ([secrets-and-supply-chain/ci-cd-attacks.md](secrets-and-supply-chain/ci-cd-attacks.md)) | SLSA + OpenSSF Scorecard + GitHub Actions hardening | least-privilege workflow tokens + pinned actions/images + isolated runners + trust-partitioned caches | workflow policy gate (`permissions`, SHA pins, no privileged untrusted PR code) → [templates/ci-gates.md](templates/ci-gates.md) |
| **Crypto** ([cryptography/](cryptography/README.md)) | NIST FIPS / SP 800-57 | a **vetted library**, no roll-your-own, strong primitives | a Semgrep "ban weak crypto" rule in CI → [templates/ci-gates.md](templates/ci-gates.md) |
| **C/C++ / embedded** ([secure-coding-standards.md](secure-coding-standards.md)) | **CERT-C/C++ + MISRA + ISO 26262** | bounded APIs + checked arithmetic + sanitizers | **cppcheck/clang-tidy gate + ASan/UBSan** in CI → [templates/ci-gates.md](templates/ci-gates.md) |

## What each gate kills (attack → control)

Enforcement is offense run backwards — every gate exists to kill a *specific attack/chain* you'd
otherwise find in [chaining-and-impact.md](chaining-and-impact.md). Keep that linkage in the report or
it's just generic DevSecOps noise:

- **mass-assignment → `isAdmin=true` → privilege escalation** ⟵ strict input schema (Web/API) ·
  [web/access-control.md](web/access-control.md)
- **SSRF → cloud metadata (IMDS) → role creds → account takeover** ⟵ IMDSv2 + egress allowlist as
  policy-as-code (Cloud) · [web/ssrf.md](web/ssrf.md)
- **indirect prompt injection → tool call → web sink (SSRF/SQLi/RCE)** ⟵ output validation + tool-arg
  allowlist (AI) · [ai-llm/prompt-injection.md](ai-llm/prompt-injection.md)
- **malicious model file (`torch.load`/pickle) → RCE on load** ⟵ safetensors + a modelscan gate ·
  [ai-llm/model-supply-chain.md](ai-llm/model-supply-chain.md)
- **leaked secret → account/cloud takeover** ⟵ pre-commit secret scan + rotation (Secrets) ·
  [secrets-and-supply-chain/secret-detection.md](secrets-and-supply-chain/secret-detection.md)
- **untrusted PR → privileged workflow token / secrets** ⟵ no `pull_request_target` code execution,
  least-privilege `permissions`, SHA-pinned actions, isolated runners (CI/CD) ·
  [secrets-and-supply-chain/ci-cd-attacks.md](secrets-and-supply-chain/ci-cd-attacks.md)
- **unbounded `strcpy`/`memcpy` → buffer overflow → RCE** ⟵ bounded APIs + a cppcheck/ASan gate (C/C++) ·
  [secure-coding-standards.md](secure-coding-standards.md)

## How to apply it (in the audit)

1. While testing a category, note not just the instance but **whether the structural control exists at
   all.** Its absence is itself the headline finding ("no central input-validation layer").
2. In the report ([reporting-and-disclosure.md](reporting-and-disclosure.md)), each class-level finding
   ends with the **structural control + the gate** (copy the template), not just the per-line patch.
3. Offer the gate as a concrete artifact — the templates are drop-in starting points the dev can wire
   into CI/pre-commit today.
4. **Name the gate's cost.** A CI gate adds build time, a validation layer adds latency, a policy gate
   can block a deploy — report this as the gate's **trade-off** ([reporting-and-disclosure.md](reporting-and-disclosure.md))
   so the owner adopts it knowingly. The trade-off is informational; it never lowers the finding's severity.

## Don't over-enforce (the FP discipline still applies)

Enforcement is a *fix*, not a finding — so the false-positive rules still hold. If the framework already
provides the control (React auto-escaping, Django ORM parameterization, a managed identity library),
**don't manufacture an "enforce X" finding** — confirm the control is present and move on. See the
guards in [pattern-triggers.md](pattern-triggers.md) and the canonical
[severity-and-triage.md](severity-and-triage.md). And state honestly what enforcement *cannot* prove
([limitations.md](limitations.md)).

## References

Standards: [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/) ·
[CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks) · [SLSA](https://slsa.dev/) ·
[NIST SSDF](https://csrc.nist.gov/Projects/ssdf) · [NIST 800-63](https://pages.nist.gov/800-63-3/) ·
[FAPI](https://openid.net/wg/fapi/) · [NIST AI RMF](https://www.nist.gov/itl/ai-risk-management-framework).
Enforcement tooling: [Semgrep](https://semgrep.dev/) · [OPA](https://www.openpolicyagent.org/) /
[Conftest](https://www.conftest.dev/) · [pre-commit](https://pre-commit.com/) ·
[OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html).
Pairs with [methodology.md](methodology.md) (the lifecycle) and the per-category leaves. Full
bibliography: [research/process.md](../research/process.md) (verification & enforcement standards).
Back to [tree](00-map.md).
