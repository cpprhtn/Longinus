# Secret detection & remediation

Hard-coded credentials (CWE-798) are the most common high-severity issue in AI-generated and
move-fast codebases. A single leaked production key can mean total compromise — and ~400+ exposed
secrets were found across one scan of ~5,600 vibe-coded apps. This is the highest-confidence,
fastest-payoff check in Longinus.

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — Hardcoded secrets in source**
```bash
rg -n -i "(api[_-]?key|secret[_-]?key|password|token|credential)\s*[:=]\s*[\"'][^\s\"']{8,}" .
```
- **SKIP if:** it's a placeholder/example value (e.g., `"your-api-key-here"`, `"changeme"`, `"xxx"`)
- **SKIP if:** path contains `/test/`, `/mock/`, `/fixtures/`, `/docs/`, `/examples/`
- **FINDING if not skipped:** Type: Hardcoded Secret | Severity: High | Fix: Move secret to environment variable or secret manager; rotate the exposed credential

**STEP 2 — High-entropy secret patterns**
```bash
rg -n "AKIA[0-9A-Z]{16}|sk_live_|ghp_[A-Za-z0-9]{36}|glpat-|xox[bpras]-|-----BEGIN.*PRIVATE KEY" .
```
- **SKIP if:** path contains `/test/`, `/mock/`, `/docs/`
- **FINDING if not skipped:** Type: Hardcoded Secret (high-confidence pattern) | Severity: Critical | Fix: Rotate immediately; move to secret manager

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## What counts as a secret

API keys, OAuth client secrets, DB connection strings/passwords, cloud access keys (AWS
`AKIA…`/secret, GCP service-account JSON, Azure), private keys (`BEGIN ... PRIVATE KEY`), JWT signing
secrets, webhook/signing secrets, SMTP creds, third-party tokens (Stripe `sk_live_…`, GitHub
`ghp_…`/`github_pat_…`, Slack `xox…`, OpenAI/Anthropic `sk-…`, Twilio, SendGrid), session/encryption
keys, and `.env`/credential files.

## Where to look (all of these)

- **Source tree** — string literals, config files, sample/`.example` files that became real.
- **Git history** — the big one. Secrets "removed" in a later commit live forever in history, in every
  clone, and in forks. Scan the *whole* history, not `HEAD`.
- **`.env` / config / cert files committed** to the repo (check `.gitignore` was set up *before* the
  first commit, not after).
- **Frontend bundles** — keys shipped to the browser (`.js`, source maps). Anything in client code is
  public by definition; only *publishable* keys belong there.
- **CI/CD** — pipeline logs printing env vars, secrets in workflow YAML, build artifacts, Docker image
  layers (`docker history`, layer inspection — secrets baked into intermediate layers persist even if
  deleted later).
- **Logs & error output** — tokens/passwords written to logs (see
  [../web/misconfiguration.md](../web/misconfiguration.md) A09).
- **Public exposure** — the org's public repos, devs' personal repos, gists, paste sites, npm/PyPI
  packages, container registries (see [../recon/osint.md](../recon/osint.md)).

## How to find it (tooling)

```bash
# fast regex + entropy + verification
gitleaks detect --source . --redact                 # working tree + history, low FP
trufflehog git file://. --only-verified             # verifies creds are LIVE (high signal)
trufflehog filesystem ./ --only-verified
# git history specifically
git log -p | rg -n -i "api[_-]?key|secret|password|token|BEGIN .*PRIVATE KEY"
# docker image layers
dive <image>    # or: docker history --no-trunc <image>
# manual high-signal patterns
rg -n -i "AKIA[0-9A-Z]{16}|sk_live_[0-9a-zA-Z]{20,}|ghp_[0-9A-Za-z]{36}|xox[baprs]-|-----BEGIN" .
```
`--only-verified` (TruffleHog) matters: it tells you which secrets are *actually live*, separating a
revoked test key (Low) from an active prod key (Critical).

## How to confirm severity (carefully)

- Determine **what the secret unlocks** and whether it's **live**. Validation should be the
  **least-privilege, read-only** call possible (e.g. an identity/whoami endpoint) — and only on
  credentials for systems **you own or are authorized to test.** For third-party finds, you generally
  *don't* validate by using it; you report it and let the owner rotate.
- Severity scales with privilege + scope: prod admin/cloud root key = Critical; revoked or test-only =
  Low/Info but still report.

> ⚠️ A found secret is not a license to access anything. Using leaked creds on systems you don't own
> exceeds authorization (see [../authorization-and-scope.md](../authorization-and-scope.md)). Validate
> minimally or not at all; disclose.

## How to fix (the right order)

1. **Rotate / revoke first.** Invalidate the exposed credential at the provider and issue a new one.
   Assume it's compromised the instant it was committed. *Do this before cleaning history* — cleanup
   doesn't un-leak.
2. **Move secrets out of code** into a secrets manager / env vars injected at runtime: Vault, AWS/GCP/
   Azure secret managers, Doppler, SOPS-encrypted files, or platform secret stores. Code reads from
   the environment; secrets never touch the repo.
3. **Purge from git history** — `git filter-repo` (preferred) or BFG, then force-push and have all
   collaborators re-clone. (Still useless without step 1.)
4. **Prevent recurrence:**
   - `.gitignore` for `.env`/keys/certs; commit a `.env.example` with *placeholders*.
   - **Pre-commit secret scanning** (gitleaks/trufflehog as a pre-commit hook) and a **CI gate** that
     fails the build on a detected secret.
   - **Push protection** (GitHub secret scanning / GitLab) on the remote.
   - scoped, short-lived, rotatable credentials; separate keys per environment; least privilege so a
     leak is contained.
5. **For client-side keys:** only ship *publishable*/restricted keys to browsers; enforce server-side
   that the secret operations require a backend. Restrict keys by referrer/IP/scope where the provider
   allows.

## Static red flags (vibe-coded patterns)

```bash
rg -n -i "const .*(key|secret|token|password)\s*=\s*['\"][^'\"]{12,}" .   # inline literal secret
rg -n "process\.env" .   # good — but verify the value isn't ALSO hard-coded as a fallback default
```
Watch for `apiKey = "sk-..." || process.env.KEY` (hard-coded fallback defeats the env var), secrets in
test fixtures, and "temporary" keys that shipped.

## CTF angle

Forensics/recon challenges frequently hide flags as "secrets": in `.git` history, Docker layers, EXIF,
env files, or JS bundles. The same tools (gitleaks, trufflehog, `git log -p`, `dive`) find them.

## References

[CWE-798](https://cwe.mitre.org/data/definitions/798.html) ·
OWASP [Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html) ·
[gitleaks](https://github.com/gitleaks/gitleaks) / [TruffleHog](https://github.com/trufflesecurity/trufflehog) docs ·
[GitHub secret scanning](https://docs.github.com/en/code-security/secret-scanning).
Full bibliography: [research/secrets-supply-chain.md](../../research/secrets-supply-chain.md). Back to [secrets-and-supply-chain/](README.md).
