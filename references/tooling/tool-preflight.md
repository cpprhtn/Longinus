# Tool preflight and Docker fallback

Run this before a serious audit when local tools may or may not exist. The goal is to make deterministic
scanners fire instead of silently falling back to LLM reasoning.

## Status model

For each tool, record one of:

| Status | Meaning |
|---|---|
| `ready` | Native command exists and can run. |
| `docker` | Native command missing, but a pinned Docker image can run it. |
| `missing` | No native command and no Docker fallback available. |
| `blocked` | Tool exists but cannot run because of auth, network, permissions, or sandbox limits. |

Report missing/blocked tools as coverage gaps, not as clean results.

## Preflight commands

```bash
command -v rg semgrep gitleaks trivy osv-scanner pip-audit govulncheck cargo-audit npm docker
```

Suggested Docker fallbacks when native tools are missing:

| Tool | Docker fallback |
|---|---|
| Semgrep | `docker run --rm -v "$PWD:/src" semgrep/semgrep semgrep --config auto /src` |
| gitleaks | `docker run --rm -v "$PWD:/repo" zricethezav/gitleaks:latest detect --source /repo --redact` |
| trivy | `docker run --rm -v "$PWD:/repo" aquasec/trivy:latest fs /repo` |
| osv-scanner | `docker run --rm -v "$PWD:/repo" ghcr.io/google/osv-scanner:latest -r /repo` |
| nuclei | `docker run --rm projectdiscovery/nuclei:latest -version` |

Prefer pinned image digests in CI. If Docker is unavailable, use the native fallback from the relevant
leaf and disclose the missed scanner.

## Output shape

```yaml
tool_preflight:
  semgrep: ready
  gitleaks: docker
  trivy: missing
  osv-scanner: blocked
```

Back to [tooling/](README.md).
