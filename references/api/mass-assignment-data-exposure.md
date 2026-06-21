# API property authorization: mass assignment and excessive data exposure

Authentication answers who the caller is. Property authorization answers which fields the caller may
write or read. Covers OWASP API3.

## Mechanical scan

```bash
# Client body bound directly into model/update
rg -n "Object\.assign\(.*req\.body|User\(\*\*request\.data\)|\.update\(req\.body\)|findByIdAndUpdate\(.*req\.body|model_validate\(.*request|from_dict\(.*request" .

# Full model serialization
rg -n "\.to_dict\(\)|\.toJSON\(\)|res\.json\(user\)|jsonify\(.*\.query\.|serialize.*all|__dict__|model_to_dict" .

# Privileged/business fields in request input
rg -n -i "req\.(body|query|params).*(role|isAdmin|permission|verified|ownerId|tenantId|balance|price|total)|request\.(data|args|json).*(role|is_admin|verified|owner_id|tenant_id|balance|price|total)" .
```

- **SKIP:** request DTO/serializer allowlists writable fields before assignment, and response DTO
  allowlists readable fields before output.
- **FINDING:** Mass Assignment | Severity: High | Fix: allowlist writable fields per role.
- **FINDING:** Excessive Data Exposure | Severity: Medium-High by data sensitivity | Fix: explicit
  response DTOs; never return raw models.

## Confirm

1. Add privileged fields to a normal request: `role`, `isAdmin`, `verified`, `ownerId`, `tenantId`,
   `balance`, `price`, `total`.
2. Read the object back or call a privileged action to prove the server accepted the field.
3. For exposure, compare raw JSON with the UI. Sensitive fields include password hashes, reset tokens,
   MFA secrets, PII, internal notes, tenant IDs, and authorization flags.

## Fix

- Use separate DTOs for create/update/read, with role-specific allowlists.
- Recompute money, totals, ownership, and authorization server-side.
- Make sensitive fields private by default at the model/serializer layer.
- Gate with tests that injected privileged fields are ignored or rejected.

## Do not report if

- The suspicious field is accepted but ignored before persistence.
- The caller is reading only their own non-sensitive profile data.
- The field is writable by design and documented for that caller role.

Back to [api/](README.md).
