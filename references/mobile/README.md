# 📱 mobile/ — Android & iOS application security

A mobile app is a client you hand to the attacker — they own the device, can decompile the binary,
intercept its traffic, and inspect its storage. The backend API is usually the bigger prize (test it
via [../api/README.md](../api/README.md)), but the app itself leaks secrets, mishandles data, and
trusts the client. Maps to **OWASP MASVS / MASTG** (the mobile equivalent of ASVS/WSTG).

> ⛔ Test apps you own/are authorized for, on your own devices/emulators.
>
> **Scope & depth.** This branch is a *navigational map* of mobile testing keyed to MASVS — enough to
> profile an app and route to the right check, not an exhaustive procedure set. For full per-test
> procedures use **[OWASP MASTG](https://mas.owasp.org/)** (the canonical guide) and
> **[HackTricks Mobile](https://book.hacktricks.wiki/)**; the backend is usually the real prize — pivot
> to [../api/README.md](../api/README.md).

> 🛡️ **Enforce-forward.** Anchor to **MASVS**. Class-killer: **platform keystore + cert pinning + no
> client-side trust**. Gate it with **MobSF/lint in CI** → [../enforce-forward.md](../enforce-forward.md).

## Mechanical scan — Mobile quick checks

Run these before the deeper analysis below.

1. Search for hardcoded keys/secrets in source / `strings` output
2. Check `AndroidManifest.xml` for exported Activities/Services/Receivers
3. Look for certificate pinning (or lack thereof) in network config
4. Check local storage for sensitive data stored unencrypted (SharedPreferences, Keychain misuse)
5. Identify the API backend → apply the [API quick checks](../api/README.md)

## Mechanical scan

> **Quick mode only.** Run these greps, apply skip conditions, report matches.
> No further analysis needed in quick mode.

**STEP 1 — Hardcoded secrets in mobile source**
```bash
rg -n -i "api[_-]?key|secret|password|token|http://" .
```
- **SKIP if:** path contains `/test/`, value is a placeholder
- **FINDING if not skipped:** Type: Hardcoded Secret in Mobile App | Severity: High | Fix: Move secrets server-side; mobile apps cannot hold secrets safely

**STEP 2 — Exported Android components**
```bash
rg -n "android:exported=\"true\"" .
```
- **SKIP if:** the component is an intended entry point (launcher activity, documented receiver)
- **FINDING if not skipped:** Type: Exported Component | Severity: Medium | Fix: Set android:exported="false" unless the component must be externally accessible

**Output template (quick mode):**
```
| File:Line | Type | Severity | Pattern | Fix |
|---|---|---|---|---|
```

## OWASP MASVS control groups (what to verify)

| MASVS group | Focus |
|---|---|
| **STORAGE** | no sensitive data in plaintext (prefs, SQLite, logs, backups, clipboard, screenshots) |
| **CRYPTO** | proper key mgmt, no hardcoded keys, no weak/custom crypto |
| **AUTH** | secure auth/session, biometric done right, no client-side-only checks |
| **NETWORK** | TLS enforced, certificate pinning, no cleartext traffic |
| **PLATFORM** | safe IPC, WebView, deep links, permissions, exported components |
| **CODE** | up-to-date deps, no debug code, safe handling of inputs |
| **RESILIENCE** | anti-tamper/RE (defense-in-depth, *not* a security boundary) |

## Workflow

1. **Get the package:** `.apk`/`.aab` (Android) or `.ipa` (iOS).
2. **Static (decompile & inspect):**
   - Android: `apktool` (resources/manifest), `jadx`/`jadx-gui` (Java/Kotlin), `MobSF` (automated).
   - iOS: `class-dump`, Hopper/Ghidra, `MobSF`.
   - Look for: **hardcoded secrets/API keys/endpoints** (the #1 finding — see
     [../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md)),
     the `AndroidManifest.xml` for **exported** activities/services/receivers/providers, permissions,
     `android:debuggable`, `allowBackup`, cleartext-traffic config, deep-link schemes.
3. **Storage review:** pull app data — `SharedPreferences`, SQLite DBs, files, `Keychain`/`Keystore`
   usage, logs, cached webviews, screenshots-on-background, clipboard. Sensitive data must be
   encrypted via the platform keystore, not rolled by hand.
4. **Dynamic & traffic:**
   - Proxy the app (Burp/mitmproxy) with a user CA; if it breaks, there's **cert pinning** — bypass on
     *your* test build/device with **Frida**/**Objection** to continue testing the API.
   - Runtime instrumentation (Frida/Objection): bypass root/jailbreak detection and client-side checks,
     hook crypto, inspect storage live.
5. **Backend is the real target:** once you can see the app's API calls, pivot to
   [../api/README.md](../api/README.md) — BOLA/BFLA, auth, mass assignment. **Never trust client-side
   controls**; everything the app "enforces" must be re-checked server-side.

## High-yield mobile findings

- **Hardcoded secrets / API keys / cloud creds** in the binary or resources.
- **Sensitive data stored in cleartext** (tokens, PII, in prefs/SQLite/logs); `allowBackup=true`
  exposing it.
- **Insecure transport** — cleartext HTTP, accepting bad certs, no pinning on sensitive flows.
- **Exported components / improper IPC** — other apps can invoke an exported activity/service/
  content-provider to read data or trigger actions; SQL injection in an exported content provider.
- **Deep-link / WebView issues** — `javascriptInterface` exposed to remote content, `file://` access,
  deep links that pass untrusted input into sensitive actions, open-redirect via custom scheme.
- **Client-side auth/logic** — "premium" or "admin" gated only in the app; enforce server-side.
- **Outdated SDKs/libs** with CVEs ([../secrets-and-supply-chain/dependency-supply-chain.md](../secrets-and-supply-chain/dependency-supply-chain.md)).

## Static red flags

```bash
# after jadx/apktool decompile:
rg -n -i "api[_-]?key|secret|password|token|http://|BEGIN .*PRIVATE KEY" .
rg -n "android:exported=\"true\"|allowBackup=\"true\"|android:debuggable|usesCleartextTraffic|addJavascriptInterface" .
```

## Confirm & fix

- **Confirm:** show the extracted secret, the readable cleartext store, the exported component invoked
  from a second app, or the unpinned/intercepted API call.
- **Fix:** keep secrets server-side (apps can't hold real secrets safely); platform keystore for any
  local sensitive data; enforce TLS + pin sensitive connections; don't export components unless needed
  (and protect with permissions/signatures); harden WebView/deep-links; **re-validate everything on the
  server**; treat RE-resistance as defense-in-depth only.

## CTF angle

Mobile CTF: decompile to find a hardcoded flag/key, reverse a license/crackme check in the smali/native
lib, bypass root detection with Frida to reach a gated function, or exploit an exported content
provider / deep link. `jadx` + `frida` cover most of it.

## References

[OWASP MASVS / MASTG](https://mas.owasp.org/) · [Frida](https://frida.re/) /
[Objection](https://github.com/sensepost/objection) /
[MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) docs ·
CWE-[312](https://cwe.mitre.org/data/definitions/312.html)/[798](https://cwe.mitre.org/data/definitions/798.html)/[926](https://cwe.mitre.org/data/definitions/926.html)/[919](https://cwe.mitre.org/data/definitions/919.html).
Full bibliography: [research/mobile.md](../../research/mobile.md). Back to [tree](../00-map.md) ·
backend → [../api/README.md](../api/README.md).
