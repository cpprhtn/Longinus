# 🔑 secrets-and-supply-chain/ — the #1 vibe-coding risk

**Start here for a vibe-coded app.** Two of the most common — and most damaging — issues in
AI-generated code are *leaked secrets* and *bad dependencies*, and both are nearly invisible to the
person who shipped the code. They're also fast to check and high-confidence, so they're the best
ROI of any branch. Maps to **OWASP Top 10:2025 A03 (Software Supply Chain Failures)** and **A08
(Software/Data Integrity Failures)**.

Why AI code is especially exposed:
- LLMs happily **hard-code API keys, DB passwords, and tokens** into examples and "just make it work"
  code, and devs commit them.
- LLMs **hallucinate package names** that don't exist — attackers pre-register them
  (*slopsquatting*) and ship malware to anyone who copies the snippet.
- LLMs suggest **outdated libraries with known CVEs** (their training predates the patch), silently
  re-introducing fixed vulnerabilities.
- Generated `Dockerfile`/CI/IaC often run as root, pin nothing, and bake in secrets.

> 🛡️ **Enforce-forward.** Anchor to **SLSA / NIST SSDF**. Class-killer: **no secrets in code + pinned
> deps + build provenance**. Gate it with **gitleaks pre-commit + an SCA gate + lockfile pinning** →
> [../enforce-forward.md](../enforce-forward.md) · [../templates/pre-commit-and-secrets.md](../templates/pre-commit-and-secrets.md) · [../templates/ci-gates.md](../templates/ci-gates.md).

## Branch map

| Leaf | Covers |
|---|---|
| [secret-detection.md](secret-detection.md) | hard-coded keys/tokens/passwords, git history, env/CI/log leakage, validation, rotation |
| [dependency-supply-chain.md](dependency-supply-chain.md) | known-CVE deps (SCA), slopsquatting/typosquatting, lockfiles, integrity, SBOM, SLSA, CI/CD & build security |

## Mechanical scan (60-second triage)

```bash
# 1) secrets in the working tree
rg -n -i "(api[_-]?key|secret|passwd|password|token|BEGIN (RSA|EC|OPENSSH) PRIVATE KEY|aws_secret|xox[baprs]-|ghp_|sk-)" .
# 2) secrets in git history (deleted ≠ gone)
gitleaks detect --source . --redact      # or: trufflehog git file://. --only-verified
# 3) committed env / credential files that shouldn't be there
git ls-files | rg -n "\.env|\.pem|\.p12|\.keystore|credentials|id_rsa|\.npmrc|\.pypirc"
# 4) known-vulnerable dependencies
npm audit --omit=dev ; pip-audit ; osv-scanner -r . ; trivy fs .
# 5) hallucinated/typosquatted packages — verify every dependency actually exists & is the real one
```

Anything live found in 1–3 is often **Critical** — treat as an active incident: rotate immediately,
then remove from history.

## Why "rotate, don't just delete"

A secret committed to git is compromised the moment it's pushed (especially to a public/forkable repo,
CI logs, or a mirror). Deleting the line does nothing — the value is in history, clones, caches, and
possibly already scraped by bots that watch public pushes. **The only fix is to rotate the credential**
(invalidate the old one at the provider), *then* purge it from history. See
[secret-detection.md](secret-detection.md).

## How this branch fits the engagement

- **Self-audit (vibe coding):** run the 60-second triage first thing; it usually finds the worst issue
  in the repo before you test a single endpoint.
- **Bug bounty / recon:** leaked secrets in the org's public repos/JS bundles are a top passive find
  (see [../recon/osint.md](../recon/osint.md)) — but *validate carefully and disclose*; don't use
  found creds to access systems (that exceeds authorization — see
  [../authorization-and-scope.md](../authorization-and-scope.md)).

## References

[OWASP Top 10:2025 A03/A08](https://owasp.org/Top10/2025/) ·
[NIST SSDF (SP 800-218)](https://csrc.nist.gov/projects/ssdf) · [SLSA](https://slsa.dev/) ·
OWASP [Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) Cheat Sheet ·
CWE-[798](https://cwe.mitre.org/data/definitions/798.html) (hard-coded creds) /
[1104](https://cwe.mitre.org/data/definitions/1104.html) (unmaintained deps) /
[1357](https://cwe.mitre.org/data/definitions/1357.html).
Full bibliography: [research/secrets-supply-chain.md](../../research/secrets-supply-chain.md). Back to [tree](../00-map.md).
