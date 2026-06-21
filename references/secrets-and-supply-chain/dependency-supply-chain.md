# Dependencies & software supply chain (A03:2025)

New as its own OWASP Top 10 category in 2025 — because most of your code is *other people's* code, and
AI assistants pull in dependencies recklessly. This covers known-vulnerable dependencies, the
AI-specific **slopsquatting** threat, and lockfile/integrity hygiene. CI/CD attack classes live in
[ci-cd-attacks.md](ci-cd-attacks.md). CWE-1104, CWE-1357, CWE-829, CWE-494.

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — Run SCA tool**
```bash
npm audit --omit=dev 2>/dev/null; pip-audit 2>/dev/null; osv-scanner -r . 2>/dev/null
```
- **SKIP if:** no package manager files found for that ecosystem
- **FINDING if not skipped:** Report each CVE found. Type: Vulnerable Dependency | Severity: Info (unless reachability confirmed → per normal triage) | Fix: Update to patched version per tool recommendation

**STEP 2 — Missing lockfile**
```bash
ls package-lock.json yarn.lock pnpm-lock.yaml poetry.lock Pipfile.lock Gemfile.lock go.sum Cargo.lock 2>/dev/null
```
- **SKIP if:** at least one lockfile exists for the detected ecosystem
- **FINDING if not skipped:** Type: Unpinned Dependencies | Severity: Medium | Fix: Generate and commit a lockfile for deterministic builds

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## 1. Known-vulnerable dependencies (SCA)

LLMs suggest libraries from their training data — often **versions with since-patched CVEs**. The fix
is to scan continuously, not once.

```bash
npm audit --omit=dev        # Node
pip-audit                   # Python   (or safety)
osv-scanner -r .            # multi-ecosystem, OSV.dev backed
trivy fs .                  # multi-ecosystem + containers + IaC
govulncheck ./...           # Go (reachability-aware)
bundler-audit               # Ruby
```
- **Prioritize by reachability and exploitability**, not raw count: a CVE in a transitively-pulled,
  never-called function is lower than a reachable one in your request path. Cross-check
  [EPSS](https://www.first.org/epss/) and whether a public exploit exists.
- **Fix:** upgrade to the patched version (mind breaking changes); if no patch, pin + mitigate +
  monitor, or replace the dependency. Automate with Dependabot/Renovate. Track everything in an
  **SBOM** (CycloneDX/SPDX) so you know what you ship.

## 2. Slopsquatting & typosquatting (the AI-era threat)

- **Slopsquatting:** LLMs **hallucinate package names that don't exist**. Attackers register those
  names on npm/PyPI and fill them with malware. A developer copies the AI's snippet, runs
  `pip install <hallucinated>`, and ships a backdoor. Studies show a meaningful fraction of
  AI-suggested packages are non-existent and the hallucinations are *repeatable* — making them
  reliably exploitable.
- **Typosquatting:** malicious packages named like popular ones (`reqursts`, `python-dateutil` vs
  `python-dateutil`-lookalikes, `electron`/`electorn`).
- **Dependency confusion:** a public package shadowing an internal one of the same name → your build
  pulls the attacker's.

**How to check (do this for every dependency in AI-generated code):**
1. **Verify the package actually exists and is the one you meant** — real registry page, expected
   maintainer/repo, sane download counts and age. Brand-new, low-download packages matching a "helpful"
   AI suggestion are suspect.
2. **Diff against the canonical name** for typos; confirm the GitHub repo the registry links to is the
   real project.
3. **Scope internal names** to your private registry and configure the installer to **prefer the
   private source** (kills dependency confusion).
4. Treat install scripts (`postinstall`, `setup.py` exec) as code — they run on `npm install`/
   `pip install`.

**Fix:** human-verify every new dependency an AI proposes; use a private registry/proxy with an
allowlist; pin to known-good versions; enable registry-side malware scanning; prefer well-maintained,
widely-used libs.

## 3. Lockfiles & integrity

- **Commit a lockfile** (`package-lock.json`, `poetry.lock`, `Cargo.lock`, `go.sum`) and **install
  from it deterministically** (`npm ci`, `pip install --require-hashes`, `--frozen-lockfile`). Without
  it, builds float to new (possibly malicious) versions.
- **Verify integrity hashes** (subresource/integrity in lockfiles, `go.sum`, hash-pinned pip).
- **Pin, but patch.** Pinning + automated update PRs (Renovate) gives reproducibility *and* timely
  fixes.
- **Subresource Integrity (SRI)** for third-party scripts loaded in the browser (`integrity=` +
  `crossorigin`) — defends against a compromised CDN (the Polyfill.io-style attack).

## 4. Build & artifact integrity (A08 overlap)

The pipeline is a prime target — compromise the build, compromise everyone downstream. For attack
patterns such as pwn-request, script injection, cache poisoning, and self-hosted runner exposure, open
[ci-cd-attacks.md](ci-cd-attacks.md). Baseline controls:
- **Least-privilege CI:** scoped tokens, no long-lived cloud admin creds in CI, OIDC federation instead
  of stored keys, secrets via the platform's secret store (not plaintext YAML).
- **Pin actions/images by digest** (`uses: actions/checkout@<sha>`, `FROM image@sha256:…`), not mutable
  tags. Untrusted third-party CI actions can exfiltrate secrets.
- **Protect the supply chain to prod:** signed commits/tags, **artifact signing** (Sigstore/cosign),
  provenance/attestation (**SLSA** levels), reproducible builds, and verification at deploy.
- **Scan IaC & images in CI** (trivy/checkov/grype) — see
  [../cloud-and-infra/iac-and-containers.md](../cloud-and-infra/iac-and-containers.md).
- **Protected branches**, mandatory review, and restricted who-can-publish on registries.

## Static / repo signs

```bash
rg -n "\"dependencies\"|require\(|^import |^from " . | head        # inventory entry points
git ls-files | rg "package-lock|yarn.lock|poetry.lock|Pipfile.lock|go.sum|Cargo.lock"  # lockfile present?
rg -n "postinstall|preinstall" package.json                       # install-time scripts
```
Red flags: no lockfile; mutable `latest`/`*` version ranges; CI using third-party actions by mutable
tag; cloud admin keys stored in CI; dependencies that don't resolve to a real, established package.

## CTF / real-world note

Supply-chain CTF/red-team scenarios: a malicious `postinstall`, a typosquatted import, a dependency-
confusion build hijack, or a compromised CI token. The defense is the same hygiene above.

## Real-world cases

- **Dependency Confusion (Alex Birsan, 2021)** — published public packages with the *same names* as
  companies' internal packages but higher version numbers; build tools preferred the public ones → code
  exec inside Apple, Microsoft, PayPal, Shopify and 30+ others ($130k+ bounties). The reason to pin
  registries/scopes and verify packages resolve to real, intended sources.
  [Writeup](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610) (Medium).
- **XZ Utils backdoor (CVE-2024-3094, 2024)** — a multi-year social-engineering campaign made a trusted
  maintainer; a malicious build artifact in `liblzma` backdoored `sshd`. Caught by luck (a 0.5s SSH
  slowdown). Lesson: trust the *build/release pipeline*, not just the source.
  [Authoritative summary](https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27) ·
  [CVE](https://nvd.nist.gov/vuln/detail/CVE-2024-3094).

## References

[OWASP Top 10:2025 A03/A08](https://owasp.org/Top10/2025/) · [SLSA](https://slsa.dev/) ·
[NIST SSDF SP 800-218](https://csrc.nist.gov/projects/ssdf) · [Sigstore/cosign](https://www.sigstore.dev/) ·
[OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/) / [OSV.dev](https://osv.dev/) ·
[CycloneDX](https://cyclonedx.org/) / [SPDX](https://spdx.dev/) (SBOM) ·
CWE-[1104](https://cwe.mitre.org/data/definitions/1104.html)/[829](https://cwe.mitre.org/data/definitions/829.html)/[494](https://cwe.mitre.org/data/definitions/494.html).
Full bibliography: [research/secrets-supply-chain.md](../../research/secrets-supply-chain.md). Back to [secrets-and-supply-chain/](README.md).
