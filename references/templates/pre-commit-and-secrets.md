# Pre-commit + secret scanning + dependency pinning

The structural control for the #1 vibe-coding bug — **leaked secrets** — and for supply-chain drift:
scan **before** code leaves the dev's machine, and pin/verify dependencies. Backs the Secrets/Supply-
chain row of [../enforce-forward.md](../enforce-forward.md). See
[../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md).

## `.pre-commit-config.yaml` (blocks secrets at commit time)

```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.4
    hooks: [{ id: gitleaks }]
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks: [{ id: detect-secrets, args: ["--baseline", ".secrets.baseline"] }]
```
```bash
pip install pre-commit && pre-commit install      # now every commit is scanned
```

## Belt-and-suspenders in CI (history, not just the diff)

```yaml
- uses: gitleaks/gitleaks-action@v2        # scans full history on PRs
  env: { GITLEAKS_LICENSE: "" }
```
> A secret found live is often **Critical** — rotate it immediately, then scrub history; deletion alone
> doesn't help ([../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md)).

## Dependency pinning & provenance (supply-chain control)

- **Commit the lockfile** (`package-lock.json` / `poetry.lock` / `Cargo.lock` / `go.sum`) and install
  with `npm ci` / `pip install -r` from a hash-pinned file.
- **Pin GitHub Actions by commit SHA**, not a moving tag (`uses: org/action@<sha>`).
- Verify every dependency actually exists & is the real package (slopsquatting/typo guard), and adopt
  **Dependabot/Renovate** + an SCA gate ([ci-gates.md](ci-gates.md)). Target **SLSA**/**SSDF** provenance
  for releases ([../secrets-and-supply-chain/dependency-supply-chain.md](../secrets-and-supply-chain/dependency-supply-chain.md)).

Back to [enforce-forward.md](../enforce-forward.md) · [templates](README.md).
