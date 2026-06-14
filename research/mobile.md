# 📱 mobile — references & tooling

Bibliography for the [`references/mobile/`](../references/mobile/README.md) branch (OWASP MASVS/MASTG,
Android/iOS). ↑ back to the [RESEARCH hub](../RESEARCH.md).

## Frameworks & standards

- **OWASP MASVS / MASTG (Mobile Application Security Verification Standard / Testing Guide)** — the
  mobile equivalent of ASVS/WSTG. → https://mas.owasp.org/

## Tooling

> Test apps you own/are authorized for, on your own devices/emulators.

- [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) (automated static+dynamic) ·
  [jadx](https://github.com/skylot/jadx) · [apktool](https://apktool.org/) (Android) ·
  [class-dump](https://github.com/nygard/class-dump) · [Hopper](https://www.hopperapp.com/) (iOS) ·
  [Frida](https://frida.re/) / [Objection](https://github.com/sensepost/objection) (instrumentation,
  pinning/root-detection bypass) · Burp + device CA.
- Ghidra/IDA for native libs → [ctf.md](ctf.md) (reverse engineering).

## Further reading (curated lists)

- [vaib25vicky/awesome-mobile-security](https://github.com/vaib25vicky/awesome-mobile-security) —
  Android & iOS security tools, guides & research.

## Related references

The backend API is usually the bigger prize → [api.md](api.md). Also [identity.md](identity.md),
[secrets-supply-chain.md](secrets-supply-chain.md) (hardcoded keys), [ctf.md](ctf.md) (RE of native
libs / crackmes), [process.md](process.md), [meta-resources.md](meta-resources.md).
