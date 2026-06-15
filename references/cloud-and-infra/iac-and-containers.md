# IaC, containers & Kubernetes

Infrastructure-as-code and containers let one bad default replicate everywhere. AI-generated
`Dockerfile`s and Terraform are notorious for running as root, pinning nothing, opening everything, and
baking in secrets. Most of this is catchable **statically, read-only** — the best kind of finding for a
vibe-coded deploy. CWE-250, CWE-269, CWE-732, CWE-16.

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — Container running as root**
```bash
rg -n "USER root|^FROM.*" --glob "Dockerfile*" .
```
- **SKIP if:** a non-root `USER` directive appears later in the same Dockerfile
- **SKIP if:** the image is used only for build/CI, never runs in production
- **FINDING if not skipped:** Type: Container Running as Root | Severity: Medium | Fix: Add a non-root USER directive in the Dockerfile

**STEP 2 — Unpinned base image**
```bash
rg -n "FROM.*:latest|FROM.*[^@]$" --glob "Dockerfile*" .
```
- **SKIP if:** pinned by `@sha256:...` digest
- **FINDING if not skipped:** Type: Unpinned Base Image (Supply Chain) | Severity: Low | Fix: Pin base image to a specific digest (@sha256:...)

**STEP 3 — Privileged container / host access**
```bash
rg -n "privileged.*true|hostPath|hostNetwork.*true|hostPID.*true" .
```
- **SKIP if:** documented justification with strong compensating controls exists
- **FINDING if not skipped:** Type: Container Escape Risk | Severity: High | Fix: Remove privileged mode; use specific capabilities instead

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## Infrastructure as Code (Terraform / CloudFormation / ARM / Pulumi)

**Scan first:**
```bash
checkov -d .            # broad IaC misconfig (TF/CFN/ARM/k8s/Dockerfile/Helm)
trivy config .          # IaC + Dockerfile misconfig + secrets
tfsec .                 # Terraform-focused
```
**High-signal patterns to confirm manually:**
- storage made public (`acl = "public-read"`, public bucket policy, missing "block public access")
- security groups / NSGs with `0.0.0.0/0` ingress on sensitive ports
- IAM with `Action = "*"`, `Resource = "*"`, or attachable admin (→ [cloud-iam.md](cloud-iam.md))
- **secrets in variables / state** — hard-coded keys, `tfvars` with creds committed, **plaintext in
  state files** (state holds resolved secrets — store it encrypted, access-controlled; never commit
  `terraform.tfstate`)
- unencrypted storage/volumes/DBs, no encryption-at-rest/in-transit (A04)
- logging/audit disabled, public RDS/snapshots, default VPC wide-open
- mutable/unpinned module & provider versions (supply chain — see
  [../secrets-and-supply-chain/dependency-supply-chain.md](../secrets-and-supply-chain/dependency-supply-chain.md))

**Fix:** least-privilege by default, encrypt everything, no public unless required + reviewed, secrets
via a secret manager (not vars/state), encrypted remote state with locking, pin modules/providers,
**policy-as-code gate in CI** (checkov/OPA/Conftest) so bad infra can't merge.

## Containers / Dockerfile

**Scan:**
```bash
trivy image <image>            # CVEs in OS+app layers, secrets, misconfig
grype <image> ; dockle <image> # vulns / best-practice lint
dive <image>                   # inspect layers for leaked secrets
```
**Common findings & fixes:**
| Finding | Risk | Fix |
|---|---|---|
| runs as `root` (no `USER`) | container escape = host risk | non-root `USER`, drop capabilities |
| `latest`/unpinned base image | irreproducible, pulls in new CVEs | pin by digest `FROM img@sha256:…` |
| secrets in image (ENV/COPY/build arg) | leak in registry/layers | build secrets via `--secret`, runtime env, never bake |
| bloated image, full OS, build tools | huge CVE surface | minimal/distroless/multi-stage, slim base |
| no healthcheck / no resource limits | DoS, instability | set limits (also at orchestrator) |
| `ADD` from URL, `curl | sh` in build | supply-chain RCE | `COPY` local, verify checksums |
| outdated packages | known CVEs | rebuild on patched base regularly |

Also: don't mount the **Docker socket** (`/var/run/docker.sock`) into containers (= host root);
don't run `--privileged`; scan images in CI and block on critical CVEs.

## Kubernetes

**Scan posture:**
```bash
kubescape scan                 # vs NSA/CIS/MITRE frameworks
kube-bench                     # CIS Kubernetes Benchmark on nodes
trivy k8s cluster              # workloads + config
# kube-hunter for active discovery (authorized clusters only)
```
**What to check:**
- **RBAC over-permissioning:** `cluster-admin` bindings, wildcard verbs/resources, service accounts
  with more than they need, `escalate`/`bind`/`impersonate` verbs, tokens auto-mounted into pods that
  don't need them.
- **Pod Security:** privileged pods, `hostPID/hostNetwork/hostIPC`, `hostPath` mounts (esp. `/` or the
  Docker socket), running as root, missing `securityContext`/`seccomp`/`AppArmor`, added capabilities.
  Enforce **Pod Security Admission** (restricted) / an admission controller (Kyverno/OPA Gatekeeper).
- **Network:** no `NetworkPolicy` (flat, any-to-any pod traffic); exposed **API server / kubelet
  (10250) / etcd (2379) / dashboard** to the internet (etcd holds all secrets unencrypted by default).
- **Secrets:** k8s Secrets are base64, **not encrypted** at rest by default — enable encryption-at-rest
  / external secret store; don't put secrets in env/ConfigMaps or image.
- **Supply chain:** unsigned/untrusted images, no admission image-verification, mutable tags.

**Fix:** least-privilege RBAC, restricted Pod Security + admission policies, default-deny
NetworkPolicies, encrypt etcd/secrets, lock down the control plane, image signing/verification, and
continuous scanning in CI and at admission.

## Container-escape surface (awareness)

Most escapes need a misconfig you can prevent: privileged containers, mounted host paths/socket,
dangerous capabilities (`CAP_SYS_ADMIN`), or a kernel CVE. The audit job is to **remove the
preconditions**, not to perform an escape on someone else's cluster.

## Static red flags (grep)

```bash
rg -n "USER root|^FROM .*:latest|--privileged|ADD http|curl.*\| *sh" .   # Dockerfiles
rg -n "0\.0\.0\.0/0|\"\\*\"|Action.*\\*|public-read|cluster-admin|hostPath|privileged: true" .
```

## CTF / range angle

CloudGoat/kube-goat/GKE-range challenges: escape a privileged/hostPath pod to the node, abuse an
over-permissioned service-account token to read secrets, or pivot via the Docker socket. The hardening
list above is the same set of doors they walk through.

## References

[CIS Docker/Kubernetes Benchmarks](https://www.cisecurity.org/cis-benchmarks) ·
[MITRE ATT&CK Containers](https://attack.mitre.org/) ·
[NSA/CISA Kubernetes Hardening Guide](https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF) ·
[trivy](https://github.com/aquasecurity/trivy) / [checkov](https://www.checkov.io/) / [kubescape](https://github.com/kubescape/kubescape) docs ·
CWE-[250](https://cwe.mitre.org/data/definitions/250.html)/[269](https://cwe.mitre.org/data/definitions/269.html)/[732](https://cwe.mitre.org/data/definitions/732.html).
Full bibliography: [research/cloud-infra.md](../../research/cloud-infra.md). Back to [cloud-and-infra/](README.md).
