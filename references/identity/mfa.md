# MFA and step-up enforcement

MFA is a server-side state machine, not a UI screen. Test whether every path that should be post-MFA is
actually gated by post-MFA server state.

## Mechanical scan

```bash
rg -n -i "mfa|totp|otp|webauthn|passkey|backup.?code|remember.?device|step.?up|two.?factor|2fa|verify.*code|challenge" .
rg -n -i "rateLimit|throttle|attempt|lockout|resend|cooldown" .
```

- **SKIP:** all sensitive routes check server-side MFA state, OTP attempts are rate-limited, and MFA
  changes require re-auth/step-up.
- **FINDING:** MFA bypass | Severity: High | Fix: enforce MFA server-side on every post-MFA route.
- **FINDING:** OTP brute force / reusable code | Severity: Medium-High | Fix: rate limit, single-use,
  short TTL, bind to session/purpose.

## Test flow

1. Log in to the pre-MFA state, then request a post-MFA endpoint directly.
2. Try legacy API/mobile endpoints that may not check MFA.
3. Test OTP replay, resend behavior, backup-code reuse, and attempt limits with tiny bounded tests.
4. Test MFA enrollment/removal after password reset; require re-auth and notify the old channel.

## Fix

- Store and enforce server-side MFA completion per session and risk context.
- Rate-limit and lock OTP attempts; make codes single-use, short-lived, and purpose-bound.
- Require re-auth/step-up before changing MFA, password, email, phone, or recovery settings.
- Treat backup codes like passwords: hashed, single-use, visible once.

Back to [identity/](README.md).
