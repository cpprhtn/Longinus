# SAML assertion validation

SAML failures are usually XML signature and condition-validation bugs. A signed document is not enough:
the application must trust the exact signed assertion it consumes.

## Mechanical scan

```bash
rg -n -i "saml|SAMLResponse|xmlsec|xml-crypto|passport-saml|onelogin|NameID|Assertion|Audience|Recipient|NotOnOrAfter|wantAssertionsSigned|wantAuthnResponseSigned" .
```

- **SKIP:** hardened library validates signed assertion ID, audience, recipient, and freshness before
  mapping identity.
- **FINDING:** Unsigned/partially signed assertion accepted | Severity: Critical | Fix: require signed
  assertion/response per profile and validate the consumed assertion.
- **FINDING:** Missing audience/recipient/time validation | Severity: High | Fix: validate all SAML
  conditions.

## Test flow

1. Identify whether the app signs/verifies the response, the assertion, or both.
2. Check XML signature wrapping risk: the app must read the signed assertion, not a sibling element.
3. Check `Audience`, `Recipient`, `Destination`, `InResponseTo`, `NotBefore`, and `NotOnOrAfter`.
4. Check parser hardening: no XXE, safe canonicalization, no comment-injection ambiguity in `NameID`.

## Fix

- Use a maintained SAML library; avoid custom XML signature code.
- Require signatures on the object the app consumes.
- Validate audience, recipient, destination, request correlation, and freshness.
- Disable external entities and harden XML parsing.

Back to [identity/](README.md).
