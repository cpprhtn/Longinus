# CI gates — fail the build when a control is missing

The enforcement layer: **SCA** (known-vuln deps), **SAST** (dangerous patterns), and **ecosystem/
language linters** wired so the pipeline **fails**, not warns. Backs every row of
[../enforce-forward.md](../enforce-forward.md). Start with a low-noise, high-signal set and ratchet up.

## SCA — dependency vulnerabilities (per ecosystem)

```bash
npm audit --omit=dev --audit-level=high        # Node
pip-audit --strict                              # Python   (or: osv-scanner -r .)
govulncheck ./...                               # Go
cargo audit --deny warnings                     # Rust
trivy fs --exit-code 1 --severity HIGH,CRITICAL .   # multi-ecosystem + IaC + containers
```
> Severity discipline: a CVE with **no confirmed reachable path** is Info/"patch anyway," not a
> blocker — see [../severity-and-triage.md](../severity-and-triage.md). Don't fail builds on noise.

## SAST — Semgrep (dangerous patterns + ban-weak-crypto)

```bash
semgrep --config=p/owasp-top-ten --config=p/secrets --error .
```
Custom rule — ban weak crypto (the Crypto row's gate):
```yaml
rules:
  - id: ban-weak-hash
    patterns:
      - pattern-either:
          - pattern: hashlib.md5(...)
          - pattern: hashlib.sha1(...)
    message: "Weak hash for security use — use SHA-256+ / a vetted KDF"
    severity: ERROR
    languages: [python]
```

## C / C++ — lint + sanitizers (the secure-coding gate)

```bash
cppcheck --enable=warning,portability --addon=misra --error-exitcode=1 src/
clang-tidy 'src/**/*.c' --warnings-as-errors='cert-*,bugprone-*'
# build the test suite with sanitizers and run it:
cc -fsanitize=address,undefined -g -O1 ... && ./tests
```
See [../secure-coding-standards.md](../secure-coding-standards.md).

## AI/LLM — scan model artifacts on load

```bash
modelscan -p ./models    # flags pickle/Keras-Lambda RCE before a model is trusted
```
See [../ai-llm/model-supply-chain.md](../ai-llm/model-supply-chain.md).

## Wire it into CI (GitHub Actions skeleton)

```yaml
name: security-gates
on: [pull_request]
jobs:
  gates:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install pip-audit && pip-audit --strict
      - uses: returntocorp/semgrep-action@v1
        with: { config: "p/owasp-top-ten p/secrets" }
      - run: trivy fs --exit-code 1 --severity HIGH,CRITICAL .
```
> Harden the workflow itself: pin actions by commit SHA, set `permissions:` least-privilege, and avoid
> `pull_request_target` with untrusted checkouts ([../secrets-and-supply-chain/dependency-supply-chain.md](../secrets-and-supply-chain/dependency-supply-chain.md)).

Back to [enforce-forward.md](../enforce-forward.md) · [templates](README.md).
