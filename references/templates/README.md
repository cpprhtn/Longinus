# 🧰 Enforcement templates

Drop-in starting points for the **gate** layer of [enforce-forward.md](../enforce-forward.md). Copy the
relevant snippet, adapt to the stack, and wire it into CI/pre-commit so the **build fails when a
structural control is missing** — that's what keeps a bug class dead.

> These are *remediation artifacts*, not findings. Only recommend a gate the project actually lacks;
> if the framework already enforces the control, say so and move on ([../pattern-triggers.md](../pattern-triggers.md)).

- [validation-layer.md](validation-layer.md) — input/output validation & sanitization layers (zod / pydantic / JSON-Schema / OpenAPI)
- [ci-gates.md](ci-gates.md) — SCA + SAST + lint build-fail gates (Semgrep, ecosystem scanners, C/C++ lint + sanitizers)
- [policy-as-code.md](policy-as-code.md) — IaC / k8s / cloud policy (checkov, tfsec, OPA / Conftest / Rego)
- [pre-commit-and-secrets.md](pre-commit-and-secrets.md) — pre-commit hooks + secret scanning + dependency pinning

Back to [enforce-forward.md](../enforce-forward.md) · [tree](../00-map.md).
