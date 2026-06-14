# Cloud IAM, storage & metadata (AWS / GCP / Azure)

Identity is the perimeter in the cloud. The highest-impact cloud findings are **public data stores**
and **IAM privilege escalation** — a single over-permissioned role or leaked key can become full
account takeover. This leaf is read-mostly enumeration + known priv-esc patterns; **test only accounts
you own or are explicitly authorized for.** CWE-732, CWE-269, CWE-284.

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
