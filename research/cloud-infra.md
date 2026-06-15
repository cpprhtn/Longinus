# ☁️ cloud-infra — references & tooling

Bibliography for the [`references/cloud-and-infra/`](../references/cloud-and-infra/README.md) branch
(IaC, containers, k8s, cloud IAM). ↑ back to the [RESEARCH hub](../RESEARCH.md).

## Frameworks & standards

- **CIS Benchmarks** (AWS/Azure/GCP/Kubernetes/Docker Foundations). →
  https://www.cisecurity.org/cis-benchmarks
- **MITRE ATT&CK (Enterprise + Cloud + Containers)** — adversary TTP matrix. →
  https://attack.mitre.org/
- **NSA/CISA Kubernetes Hardening Guide.** →
  https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF
- **HackTricks Cloud** — practical cloud/pentest reference. → https://cloud.hacktricks.wiki/
- **Rhino Security Labs AWS IAM privilege-escalation research** (the priv-esc table in `cloud-iam.md`). →
  https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/

## Tooling

> Dual-use: run only in scope (see [process.md](process.md) → authorization). Cloud-exploitation tools
> need explicit authorization.

- IaC/container static: [Checkov](https://www.checkov.io/) · [Trivy](https://github.com/aquasecurity/trivy) ·
  [tfsec](https://github.com/aquasecurity/tfsec) · [KICS](https://kics.io/) ·
  [Dockle](https://github.com/goodwithtech/dockle)
- Cloud posture (read-only): [Prowler](https://github.com/prowler-cloud/prowler) ·
  [ScoutSuite](https://github.com/nccgroup/ScoutSuite) ·
  [CloudFox](https://github.com/BishopFox/cloudfox) · [Steampipe](https://steampipe.io/)
- K8s: [kubescape](https://github.com/kubescape/kubescape) ·
  [kube-bench](https://github.com/aquasecurity/kube-bench) ·
  [kube-hunter](https://github.com/aquasecurity/kube-hunter) ·
  [Kyverno](https://kyverno.io/) · [OPA Gatekeeper](https://open-policy-agent.github.io/gatekeeper/) ·
  [Conftest](https://www.conftest.dev/)
- Cloud exploitation (authorized): [Pacu](https://github.com/RhinoSecurityLabs/pacu) ·
  practice: [CloudGoat](https://github.com/RhinoSecurityLabs/cloudgoat) · [flAWS](http://flaws.cloud/)

## Further reading (curated lists)

- [4ndersonLin/awesome-cloud-security](https://github.com/4ndersonLin/awesome-cloud-security) ·
  [Funkmyster/awesome-cloud-security](https://github.com/Funkmyster/awesome-cloud-security) — cloud
  security tools, standards, blogs & projects.

## Related references

[secrets-supply-chain.md](secrets-supply-chain.md) (image/registry, IaC secrets) · [web.md](web.md)
(SSRF → metadata creds) · [process.md](process.md) · [meta-resources.md](meta-resources.md).
