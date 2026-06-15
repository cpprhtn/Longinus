# Cloud IAM, storage & metadata (AWS / GCP / Azure)

Identity is the perimeter in the cloud. The highest-impact cloud findings are **public data stores**
and **IAM privilege escalation** — a single over-permissioned role or leaked key can become full
account takeover. This leaf is read-mostly enumeration + known priv-esc patterns; **test only accounts
you own or are explicitly authorized for.** CWE-732, CWE-269, CWE-284.

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — IAM wildcard policies**
```bash
rg -n "Action.*\"\*\"|Resource.*\"\*\"|Effect.*Allow.*Action.*\*" .
```
- **SKIP if:** it's a `Deny` policy or scoped by a tight `Condition` block
- **FINDING if not skipped:** Type: Over-Privileged IAM | Severity: High | Fix: Apply least-privilege; replace wildcards with specific actions/resources

**STEP 2 — Public storage exposure**
```bash
rg -n "public-read|allUsers|allAuthenticatedUsers|BlockPublicAccess.*false|block_public_access.*false" .
```
- **SKIP if:** the bucket intentionally serves public static assets with no sensitive data
- **FINDING if not skipped:** Type: Public Storage Exposure | Severity: High | Fix: Enable block-public-access; use signed URLs for private content

**STEP 3 — IMDSv1 enabled**
```bash
rg -n "http_tokens.*optional|HttpTokens.*optional|metadata_options|instance_metadata" .
```
- **SKIP if:** `http_tokens = "required"` is set (IMDSv2 enforced)
- **FINDING if not skipped:** Type: IMDSv1 Enabled → Credential Theft via SSRF | Severity: Critical | Fix: Require IMDSv2 (http_tokens = "required")

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## 1. Exposed storage (the classic leak)

- **AWS S3 / GCP GCS / Azure Blob** public read or **write** (write is worse — defacement/malware
  hosting/supply-chain). Check bucket policies, ACLs, "block public access" / uniform bucket-level
  access, and signed-URL leakage.
- **Find:** enumerate buckets tied to the org (naming patterns, grayhatwarfare-style search,
  certificate/JS references). For owned/authorized accounts: `prowler`/`scoutsuite` flags public
  storage directly.
- **Confirm (non-destructive):** an anonymous `HEAD`/list proving public access — don't bulk-download
  third-party data; note it and report.
- **Fix:** block public access at the account level, least-privilege bucket policies, no public ACLs,
  encrypt at rest, short-lived scoped signed URLs, access logging.

## 2. IAM enumeration & privilege escalation

Once you have *any* credential (your own test key, or one obtained via SSRF/leak in an authorized
test), enumerate what it can do, then look for a path to more.

**Enumerate (read-only):**
```bash
aws sts get-caller-identity              # who am I
aws iam get-account-authorization-details # policies (if permitted)
# tooling:
cloudfox aws all-checks                  # surfaces priv-esc paths & exposures
pacu                                     # AWS exploitation framework (authorized only)
scoutsuite aws|gcp|azure                 # posture
```

**Known AWS priv-esc patterns (and the permission that enables each):**
- `iam:CreatePolicyVersion` / `iam:SetDefaultPolicyVersion` → rewrite a policy you're attached to →
  admin.
- `iam:PassRole` + `ec2:RunInstances` / `lambda:CreateFunction` / `glue`/`cloudformation` → run code
  as a more-privileged role.
- `iam:AttachUserPolicy` / `PutUserPolicy` / `AddUserToGroup` → attach `AdministratorAccess`.
- `iam:CreateAccessKey` (on another user) → impersonate.
- `lambda:UpdateFunctionCode` on a privileged function; `sts:AssumeRole` on an over-trusting role.
- wildcards (`Action:*`, `Resource:*`) and overly broad trust policies anywhere.

GCP analogs: `iam.serviceAccounts.actAs` / `getAccessToken`, `setIamPolicy`,
`deploymentmanager`/`cloudfunctions` as a privileged SA, editor/owner sprawl. Azure: `Owner`/`User
Access Administrator`, Managed Identity abuse, `Microsoft.Authorization/roleAssignments/write`,
Automation/Runbook as a privileged identity.

**Confirm:** demonstrate the escalation *path* (the permissions exist and chain) on your own/authorized
account — you don't have to fully execute destructive escalation to prove it.

**Fix:** least privilege (no wildcards), scope `PassRole`/`actAs` tightly, separate duties, permission
boundaries / SCPs / org policies, no long-lived keys (prefer roles/OIDC federation/workload identity),
rotate & monitor, deny by default, regular access reviews, and automated priv-esc-path detection
(cloudfox/Access Analyzer).

## 3. Instance metadata → credential theft (SSRF chain)

The most impactful cloud chain in the wild: app **SSRF** ([../web/ssrf.md](../web/ssrf.md)) → instance
metadata → role credentials → IAM access.
- AWS: `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>` (IMDSv1). **Fix:**
  enforce **IMDSv2** (session token, hop-limit=1), least-privilege instance role, restrict metadata
  egress.
- GCP: `http://metadata.google.internal/computeMetadata/v1/...` (needs `Metadata-Flavor: Google`).
- Azure: `http://169.254.169.254/metadata/identity/oauth2/token` (needs `Metadata: true`).
- **Fix everywhere:** patch the SSRF, harden metadata (v2/required headers), and minimize the
  attached identity's permissions so a stolen token is low-value.

## 4. Serverless & managed services

- **Lambda/Functions:** env-var secrets, over-permissioned execution role, event-source injection, and
  code from untrusted sources. Least-privilege the role; secrets in a secret manager; validate event
  input.
- **Exposed dashboards/APIs:** Kubernetes API, databases (RDS/Cosmos/BigQuery) public, message queues,
  ElastiCache/Redis, Elasticsearch/OpenSearch open. Lock to private networks + auth.
- **Cross-account trust:** over-trusting `AssumeRole`/resource policies allowing external principals.

## 5. Logging & detection (A09)

- **CloudTrail / GCP Audit Logs / Azure Activity** enabled, centralized, immutable; GuardDuty/Defender/
  SCC on; alerting on root use, policy changes, new keys, anomalous API calls. Absence = the breach is
  invisible and is itself a finding.

## DO NOT report as a cloud IAM vulnerability if…

- The `Action: "*"` is in a **Deny** policy or scoped by a tight `Condition` key
- The "public bucket" **intentionally serves public static assets** (website hosting) with
  no sensitive data and a documented justification
- The "over-privileged role" is a **break-glass / emergency** role with compensating
  controls (MFA-protected, logged, alarmed, restricted to a specific human)
- The "IMDSv1" finding is on an instance that has **no SSRF-vulnerable service** running
  and no outbound network path — still recommend v2, but severity is Low
- The permissions are granted to a **Service Control Policy (SCP)** boundary that
  further restricts them — the effective permissions may be narrower than they appear

---

## Vulnerable ↔ fixed IaC examples

### Over-privileged IAM role (the most common cloud finding)

```hcl
# ❌ VULNERABLE — wildcard admin on a Lambda execution role
resource "aws_iam_role_policy" "lambda_policy" {
  role = aws_iam_role.lambda_role.id
  policy = jsonencode({
    Statement = [{
      Effect   = "Allow"
      Action   = "*"              # full admin — blast radius is entire account
      Resource = "*"
    }]
  })
}

# ✅ FIXED — least-privilege scoped to what the function actually needs
resource "aws_iam_role_policy" "lambda_policy" {
  role = aws_iam_role.lambda_role.id
  policy = jsonencode({
    Statement = [{
      Effect   = "Allow"
      Action   = ["dynamodb:GetItem", "dynamodb:PutItem"]
      Resource = "arn:aws:dynamodb:us-east-1:123456789012:table/orders"
    }]
  })
}
```

### Public S3 bucket

```hcl
# ❌ VULNERABLE — public read on a data bucket
resource "aws_s3_bucket_acl" "data" {
  bucket = aws_s3_bucket.data.id
  acl    = "public-read"          # world-readable — data breach waiting to happen
}

# ✅ FIXED — block all public access at bucket level
resource "aws_s3_bucket_public_access_block" "data" {
  bucket                  = aws_s3_bucket.data.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

### IMDSv1 enabled (SSRF → credential theft)

```hcl
# ❌ VULNERABLE — IMDSv1 allows unauthenticated metadata access
# One SSRF = full role credentials
resource "aws_instance" "app" {
  metadata_options {
    http_tokens = "optional"      # IMDSv1 still works alongside v2
  }
}

# ✅ FIXED — require IMDSv2 (token-based, not reachable via simple SSRF redirect)
resource "aws_instance" "app" {
  metadata_options {
    http_tokens                 = "required"   # IMDSv2 only
    http_put_response_hop_limit = 1            # prevent container-to-host relay
  }
}
```

---

## Static red flags (IaC/config)

```bash
rg -n "\"Action\"\s*:\s*\"\\*\"|Resource.*\\*|PassRole|AssumeRole|public-read|allUsers|allAuthenticatedUsers|0\.0\.0\.0/0" .
rg -n "http_tokens\s*=\s*\"optional\"|metadata_options" .   # AWS IMDSv1 allowed
```

## CTF / cloud-range angle

flAWS, CloudGoat, IAM Vulnerable: anonymous S3 read → find a key → `iam:CreatePolicyVersion` or
`PassRole`+`RunInstances` to escalate → read the flag; or SSRF → metadata creds → enumerate → escalate.
The priv-esc table above is the cheat sheet.

## References

[CIS AWS/GCP/Azure Foundations Benchmarks](https://www.cisecurity.org/cis-benchmarks) ·
[MITRE ATT&CK Cloud](https://attack.mitre.org/) · provider IAM best-practices ·
[HackTricks Cloud](https://cloud.hacktricks.wiki/) ·
[Rhino Security IAM priv-esc research](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/) ·
CWE-[732](https://cwe.mitre.org/data/definitions/732.html)/[269](https://cwe.mitre.org/data/definitions/269.html)/[284](https://cwe.mitre.org/data/definitions/284.html).
Full bibliography: [research/cloud-infra.md](../../research/cloud-infra.md). Back to [cloud-and-infra/](README.md).
