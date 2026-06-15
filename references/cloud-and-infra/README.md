# ☁️ cloud-and-infra/ — cloud, IaC & containers

Vibe-coded apps get deployed to the cloud by people who've never seen an IAM policy. The result:
public buckets, `0.0.0.0/0` security groups, containers running as root, admin keys in CI, and
Terraform that ships all of the above as code. Often the *infrastructure* is softer than the app.
Maps to **OWASP A02:2025 (Security Misconfiguration)**, **CIS Benchmarks**, and **MITRE ATT&CK
(Cloud/Containers)**.

> ⛔ Gate hard. Cloud testing can trip provider abuse rules and hit shared infrastructure. Test only
> accounts/resources you own or are explicitly authorized for; default to **config review** (read-only)
> over live exploitation.

## Mechanical scan — Cloud/IaC quick checks

Run these before the deeper config review below.

1. `rg "Action.*\\*|Resource.*\\*|public-read|allUsers|0\\.0\\.0\\.0/0"` → immediate red flags
2. Check for IMDSv1 (`http_tokens = "optional"` or missing metadata block)
3. Check Dockerfiles for `USER root` / unpinned base images / secrets in build args
4. Check for secrets in `.tf` / `tfvars` / `docker-compose.yml` / env files
5. Check k8s manifests for `privileged: true` / `hostPath` / `cluster-admin` binding

## Branch map

| Leaf | Covers |
|---|---|
| [iac-and-containers.md](iac-and-containers.md) | Terraform/CloudFormation, Dockerfile, Kubernetes, image & IaC scanning, container escape surface |
| [cloud-iam.md](cloud-iam.md) | AWS/GCP/Azure IAM misconfig & privilege escalation, exposed storage, metadata/SSRF, logging |

## Two modes

1. **Config audit (preferred, read-only):** review IaC and cloud config against CIS Benchmarks and
   known bad patterns. Fast, safe, comprehensive, and exactly what a vibe-coded deploy needs. Tools:
   `checkov`, `tfsec`/`trivy config`, `kube-bench`, `prowler`/`scoutsuite` (read-only posture scan),
   `kubescape`.
2. **Cloud pentest (authorized):** from a foothold (a key, an SSRF, a compromised pod), enumerate and
   attempt privilege escalation and lateral movement — strictly within authorized scope and
   non-destructive. Tools: `pacu` (AWS), `ScoutSuite`, `cloudfox`, `kube-hunter`.

## 60-second cloud-posture triage

```bash
# IaC & container config (static, safe)
checkov -d .                 # Terraform/CFN/k8s/Dockerfile misconfig
trivy config .               # IaC misconfig + Dockerfile
trivy image <image>          # image CVEs + secrets + misconfig
kubescape scan               # k8s posture vs frameworks
# cloud account posture (read-only; needs read creds you own)
prowler aws                  # or: scoutsuite aws|gcp|azure
```

## The usual suspects (check these first)

- **Public object storage** — S3/GCS/Azure blobs world-readable or world-writable; this is the classic
  data-leak. Verify bucket policies/ACLs and "block public access".
- **Over-permissive network** — security groups / firewall rules open to `0.0.0.0/0` on admin ports
  (22, 3389, 5432, 6379, 9200, 27017…); unauthenticated DBs/caches exposed.
- **IAM sprawl** — wildcard `Action:*`/`Resource:*`, `*PassRole`, attachable admin policies → privilege
  escalation ([cloud-iam.md](cloud-iam.md)).
- **Secrets in config / CI** — keys in Terraform vars, env, user-data, container layers (see
  [../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md)).
- **Metadata exposure** — IMDSv1 enabled → SSRF steals instance creds
  ([../web/ssrf.md](../web/ssrf.md), [cloud-iam.md](cloud-iam.md)).
- **No logging/monitoring** — CloudTrail/Audit logs off, no alerting (A09 — you won't see the breach).
- **Containers as root**, no resource limits, privileged pods, mounted Docker socket
  ([iac-and-containers.md](iac-and-containers.md)).
- **Unpatched managed services / exposed dashboards** — Kubernetes API/dashboard, etcd, kubelet,
  Grafana/Jenkins/Argo open to the internet.

## How this fits

For a **vibe-coding self-audit**, run the config-audit triage right after the secrets check — it's the
same "fast, high-confidence, you own it" category and catches the deployment mistakes the app review
won't. For **authorized pentest**, infra is a prime pivot: SSRF → metadata creds → IAM priv-esc →
account takeover is one of the most impactful real-world chains.

## CTF / cloud-range angle

Cloud CTFs (flAWS, CloudGoat, simulators) drill exactly these: anonymous S3 read, IAM enumeration and
privilege escalation (`iam:PassRole`+`ec2`, policy version rollback, Lambda role abuse), SSRF-to-
metadata, and Lambda/serverless misconfig. The methodology here maps directly.

## References

[CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks) ·
[MITRE ATT&CK Cloud/Containers](https://attack.mitre.org/) · [OWASP A02:2025](https://owasp.org/Top10/2025/) ·
[HackTricks Cloud](https://cloud.hacktricks.wiki/) · provider security best-practice docs.
Full bibliography: [research/cloud-infra.md](../../research/cloud-infra.md). Back to [tree](../00-map.md).
