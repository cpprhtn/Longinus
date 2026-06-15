# 🔑 secrets-supply-chain — references & tooling

Bibliography for the [`references/secrets-and-supply-chain/`](../references/secrets-and-supply-chain/README.md)
branch (OWASP A03/A08 + the #1 vibe-coding risk). ↑ back to the [RESEARCH hub](../RESEARCH.md).

## Frameworks & standards

- **OWASP Top 10:2025 A03 (Software Supply Chain Failures)** + **NIST SSDF (SP 800-218)**. →
  https://owasp.org/Top10/2025/ · https://csrc.nist.gov/projects/ssdf
- **SLSA (Supply-chain Levels for Software Artifacts)** — build-integrity framework. → https://slsa.dev/
- **Sigstore** (cosign artifact signing) + **in-toto** (provenance/attestation). →
  https://www.sigstore.dev/ · https://in-toto.io/
- **SBOM formats:** CycloneDX → https://cyclonedx.org/ ; SPDX → https://spdx.dev/
- **OSV.dev** (open vulnerability DB) + **OSV-Scanner**. → https://osv.dev/
- **Slopsquatting / hallucinated dependencies** — AI suggests non-existent packages an attacker can
  register. Evidence in [rationale.md](rationale.md) (AI-code risk).
- **OWASP Secrets Management & Software Supply Chain Cheat Sheets.** →
  https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html

## Tooling

> Dual-use: run only in scope (see [process.md](process.md) → authorization).

- Secrets: [gitleaks](https://github.com/gitleaks/gitleaks) ·
  [TruffleHog](https://github.com/trufflesecurity/trufflehog) ·
  [dive](https://github.com/wagoodman/dive) (image layers) ·
  [GitHub secret scanning](https://docs.github.com/en/code-security/secret-scanning)
- SCA/deps: [osv-scanner](https://github.com/google/osv-scanner) ·
  [Trivy](https://github.com/aquasecurity/trivy) · [Grype](https://github.com/anchore/grype) ·
  [pip-audit](https://github.com/pypa/pip-audit) ·
  [govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck) ·
  [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/) ·
  [Dependabot](https://github.com/dependabot) · [Renovate](https://github.com/renovatebot/renovate)
- SBOM/integrity: [Syft](https://github.com/anchore/syft) · [cosign / Sigstore](https://www.sigstore.dev/) ·
  [git-filter-repo](https://github.com/newren/git-filter-repo) · [BFG](https://rtyley.github.io/bfg-repo-cleaner/)

## Related references

[rationale.md](rationale.md) (why this is first for AI-generated code) · [web.md](web.md) (A08
deserialization/integrity) · [cloud-infra.md](cloud-infra.md) (image/registry) · [recon.md](recon.md)
(leaked-secret discovery) · [process.md](process.md) · [meta-resources.md](meta-resources.md).
