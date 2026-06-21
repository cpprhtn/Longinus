# CI/CD attack surface

Build pipelines are production code with secrets. A malicious pull request, mutable action, poisoned
cache, or self-hosted runner can turn CI into code execution with repository/cloud privileges.

## Mechanical scan

```bash
# GitHub Actions and common CI definitions
rg -n -i "pull_request_target|workflow_run|actions/github-script|github\.context|run:.*\$\{\{|uses: .*@v[0-9]+|uses: .*@main|uses: .*@master|actions/cache|self-hosted|GITHUB_TOKEN|secrets\.|id-token|permissions:" .github .gitlab-ci.yml Jenkinsfile azure-pipelines.yml CircleCI 2>/dev/null

# Shelling out with PR-controlled data
rg -n -i "github\.event\.pull_request|github\.head_ref|github\.ref|github\.actor|inputs\.|matrix\.|run:.*(echo|bash|sh|python|node).*\\$\\{\\{" .github 2>/dev/null
```

- **SKIP:** workflow runs only on trusted branches, actions/images are pinned by SHA/digest, token
  permissions are least-privilege, and untrusted PR data is never interpolated into shell/script.
- **FINDING:** `pull_request_target` checks out or runs PR code | Severity: High-Critical | Fix: do
  not run untrusted code with privileged context.
- **FINDING:** Script injection via `${{ github.* }}` / inputs in `run:` | Severity: High | Fix: pass
  data through env vars and quote/validate.
- **FINDING:** Mutable action/image or poisoned cache path | Severity: Medium-High | Fix: pin and
  scope caches.
- **FINDING:** Self-hosted runner exposed to untrusted PRs | Severity: Critical when secrets/network
  are reachable | Fix: isolate or disallow untrusted jobs.

## High-yield checks

- **pwn request:** `pull_request_target` has write token/secrets and checks out attacker code.
- **Script injection:** untrusted PR title/body/branch/input is interpolated into shell or
  `actions/github-script`.
- **Cache poisoning:** untrusted job writes a cache later restored by a privileged job.
- **Mutable supply chain:** actions pinned to tags/branches instead of full SHAs; containers use
  `latest`.
- **Overbroad token:** missing `permissions:` means defaults may be too powerful.
- **Self-hosted runner poisoning:** untrusted PR code runs on a persistent runner with network/secrets.

## Fix

- Use `pull_request` for untrusted code; reserve `pull_request_target` for metadata-only workflows.
- Set `permissions:` to least privilege per job; prefer OIDC federation over stored cloud keys.
- Pin third-party actions by full commit SHA and images by digest.
- Never interpolate untrusted context directly into shell; pass through quoted env vars.
- Partition caches by trust level; never share caches from untrusted to privileged jobs.
- Use ephemeral isolated runners for untrusted code; protect self-hosted runners.

Back to [secrets-and-supply-chain/](README.md).
