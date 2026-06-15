# Policy-as-code — the Cloud/IaC enforcement gate

The class-killer for cloud misconfig is **private-by-default + least-privilege enforced as code**:
scan IaC before apply and **deny** the dangerous patterns in CI. Backs the Cloud/IaC row of
[../enforce-forward.md](../enforce-forward.md). See findings in
[../cloud-and-infra/iac-and-containers.md](../cloud-and-infra/iac-and-containers.md) /
[../cloud-and-infra/cloud-iam.md](../cloud-and-infra/cloud-iam.md).

## Off-the-shelf scanners (fastest start)

```bash
checkov -d . --compact --hard-fail-on HIGH        # Terraform/CFN/k8s/Dockerfile, CIS-mapped
tfsec .                                            # Terraform-focused
trivy config --exit-code 1 --severity HIGH,CRITICAL .
```
These map to **CIS Benchmarks** out of the box — adopt them as the baseline gate.

## Custom policy — OPA / Conftest (Rego) for your own rules

`policy/deny.rego`:
```rego
package main

# Deny public S3 buckets
deny[msg] {
  input.resource.aws_s3_bucket[name].acl == "public-read"
  msg := sprintf("S3 bucket '%s' is public-read", [name])
}

# Require IMDSv2 on EC2 (blocks the SSRF→credential-theft class)
deny[msg] {
  ec2 := input.resource.aws_instance[name]
  not ec2.metadata_options.http_tokens == "required"
  msg := sprintf("EC2 '%s' must require IMDSv2 (http_tokens=required)", [name])
}

# Deny privileged containers (k8s)
deny[msg] {
  c := input.spec.containers[_]
  c.securityContext.privileged == true
  msg := sprintf("container '%s' runs privileged", [c.name])
}
```
Run it on the plan / manifests:
```bash
terraform plan -out=tf.plan && terraform show -json tf.plan > plan.json
conftest test plan.json --policy policy/
conftest test k8s/ --policy policy/      # yaml manifests
```

## Wire into CI

```yaml
- run: checkov -d . --hard-fail-on HIGH
- run: |
    terraform show -json tf.plan > plan.json
    conftest test plan.json --policy policy/
```
> Pair with least-privilege CI credentials (OIDC, scoped roles) so the pipeline itself isn't the weak
> link ([../cloud-and-infra/cloud-iam.md](../cloud-and-infra/cloud-iam.md)).

Back to [enforce-forward.md](../enforce-forward.md) · [templates](README.md).
