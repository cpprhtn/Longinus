# 🔗 Cloud-account takeover chains

Where a "medium" web bug becomes company-ending: the chain **jumps planes** from the application into
the cloud control plane. The crown-jewel pivot of modern offense. Parent:
[../chaining-and-impact.md](../chaining-and-impact.md).

## Chain 1 — SSRF → IMDS → role creds → IAM priv-esc → account takeover

1. **SSRF** in a fetch feature (webhook, URL import, PDF/preview) → [../web/ssrf.md](../web/ssrf.md).
2. Point it at the **instance metadata service** (AWS `169.254.169.254`, GCP `metadata.google.internal`,
   Azure) → steal the **instance role's temporary credentials**.
3. Use the creds; **enumerate + escalate IAM** (`iam:PassRole`+compute, policy-version rollback, Lambda
   role abuse) → [../cloud-and-infra/cloud-iam.md](../cloud-and-infra/cloud-iam.md).
4. Reach S3/secrets/other tenants → **full account takeover**.
- This is the **Capital One (2019)** chain (SSRF→IMDSv1→S3, ~106M records). The reason IMDSv2 exists.

## Chain 2 — app RCE → container escape → host → cluster

1. Any **RCE** primitive in the app (see [rce-chains.md](rce-chains.md)).
2. **Escape the container:** privileged container, mounted `docker.sock`, `hostPath`, `CAP_SYS_ADMIN`,
   or a writable host mount → [../cloud-and-infra/iac-and-containers.md](../cloud-and-infra/iac-and-containers.md).
3. On the node, steal the **kubelet/service-account token** → query the API server → **cluster-wide**
   pivot (other pods' secrets, other tenants).

## Chain 3 — CI/CD → OIDC trust → cloud

1. A poisoned dependency / `pull_request_target` / injected workflow step runs in **CI** →
   [../secrets-and-supply-chain/dependency-supply-chain.md](../secrets-and-supply-chain/dependency-supply-chain.md).
2. The CI job holds a **cloud OIDC trust** (or long-lived deploy keys) → assume the deploy role →
   **production cloud access**. The pipeline is often softer than the app.

## Real-world cases

[Capital One SSRF→IMDS writeup](https://blog.appsecco.com/an-ssrf-privileged-aws-keys-and-the-capital-one-breach-4c3c2cded3af)
· [TOP SSRF (HackerOne)](https://github.com/reddelexc/hackerone-reports/blob/master/docs/tops_by_bug_type/TOPSSRF.md)
— cloud-metadata chains dominate. See [../web/ssrf.md](../web/ssrf.md) "Real-world cases."

## Prove each link

Stop at **reachability** on targets you don't own: *"IMDS is reachable via this SSRF — here's the role
name"* is the PoC; **do not** loot the bucket or use the creds ([../authorization-and-scope.md](../authorization-and-scope.md)).
Run the full chain only on your own infra/lab.

## 🛡️ Defensive seal

- **Enforce IMDSv2 (`http_tokens=required`) + least-privilege instance roles + egress allowlist** → Chain 1 dies even with SSRF.
- **No privileged containers / no `docker.sock` / drop caps / read-only rootfs; scoped service-account tokens** → Chain 2.
- **Short-lived OIDC with tight `sub` conditions; pin actions by SHA; no secrets in CI logs** → Chain 3.

→ as **policy-as-code** gates: [../enforce-forward.md](../enforce-forward.md) (Cloud row) · [../templates/policy-as-code.md](../templates/policy-as-code.md).

Back to [chain catalog](../chaining-and-impact.md) · [tree](../00-map.md).
