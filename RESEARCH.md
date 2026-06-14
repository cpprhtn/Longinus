# 📚 RESEARCH — bibliography hub

The research behind Longinus: the frameworks each branch is built on, the design rationale, and every
external source with a link. **This is an index, not a document to read top-to-bottom** — like
[references/00-map.md](references/00-map.md) is for the playbooks, this routes you to the *one* domain
bibliography you need. Each leaf in the tree links straight to its matching `research/<domain>.md`.

> **Living document.** References drift (versions, URLs, ownership). Before relying on a specific
> control, version, or flag, confirm against the canonical link. Policy: [research/meta-resources.md](research/meta-resources.md).

---

## Jump to a domain bibliography

| Domain | Frameworks + tooling | Playbook branch |
|---|---|---|
| 🔭 Recon / OSINT | [research/recon.md](research/recon.md) | [references/recon/](references/recon/README.md) |
| 🌐 Web | [research/web.md](research/web.md) | [references/web/](references/web/README.md) |
| 🔌 API | [research/api.md](research/api.md) | [references/api/](references/api/README.md) |
| 🪪 Identity (OAuth/JWT/SAML) | [research/identity.md](research/identity.md) | [references/identity/](references/identity/README.md) |
| 🔑 Secrets & supply chain | [research/secrets-supply-chain.md](research/secrets-supply-chain.md) | [references/secrets-and-supply-chain/](references/secrets-and-supply-chain/README.md) |
| 🤖 AI / LLM / agent | [research/ai-llm.md](research/ai-llm.md) | [references/ai-llm/](references/ai-llm/README.md) |
| ☁️ Cloud & infra | [research/cloud-infra.md](research/cloud-infra.md) | [references/cloud-and-infra/](references/cloud-and-infra/README.md) |
| 📱 Mobile | [research/mobile.md](research/mobile.md) | [references/mobile/](references/mobile/README.md) |
| 🧩 CTF (pwn/reverse/crypto/forensics) | [research/ctf.md](research/ctf.md) | [binary](references/binary-exploitation/README.md) · [reverse](references/reverse-engineering/README.md) · [crypto](references/cryptography/README.md) · [forensics](references/forensics/README.md) |
| ⛓️ Methodology / severity / disclosure | [research/process.md](research/process.md) | [spine](references/00-map.md) |

**Cross-cutting:** [research/rationale.md](research/rationale.md) (the idea + AI-code-risk evidence +
CTF taxonomy) · [research/meta-resources.md](research/meta-resources.md) (payload encyclopedias,
wordlists, distros, practice ranges, living-doc policy).

---

## Framework → where it lives (cross-reference)

| Standard / framework | Bibliography | Applied in |
|---|---|---|
| OWASP Top 10:2025 | [research/web.md](research/web.md) | `web/` (A01–A10 in `web/README.md`) |
| OWASP API Security Top 10:2023 | [research/api.md](research/api.md) | `api/README.md` |
| OWASP LLM Top 10:2025 · MITRE ATLAS · NIST AI RMF | [research/ai-llm.md](research/ai-llm.md) | `ai-llm/` |
| OWASP MASVS / MASTG | [research/mobile.md](research/mobile.md) | `mobile/README.md` |
| OWASP WSTG / ASVS / Cheat Sheets | [research/web.md](research/web.md) | `web/`, `api/`, `identity/` |
| OAuth / OIDC / JWT RFCs | [research/identity.md](research/identity.md) | `identity/` |
| NIST SSDF · SLSA · Sigstore · OSV · SBOM | [research/secrets-supply-chain.md](research/secrets-supply-chain.md) | `secrets-and-supply-chain/` |
| CIS Benchmarks · MITRE ATT&CK · NSA-CISA k8s | [research/cloud-infra.md](research/cloud-infra.md) | `cloud-and-infra/` |
| CWE · CAPEC · CVSS 4.0 · EPSS · KEV · SSVC | [research/process.md](research/process.md) | `severity-and-triage.md` |
| PTES · NIST 800-115 · WSTG | [research/process.md](research/process.md) | `methodology.md` |
| RFC 9116 · ISO 29147/30111 · CVD · CFAA/CMA | [research/process.md](research/process.md) | `authorization-and-scope.md`, `reporting-and-disclosure.md` |
| CTF Wiki · pwn.college · CryptoHack · Cryptopals | [research/ctf.md](research/ctf.md) | the four CTF branches |

---

## How this bibliography is organized

- Every `research/<domain>.md` holds the **frameworks + canonical tool URLs** for one branch, and
  cross-links to sibling domains and back here.
- Each playbook leaf's "References" footer links to its domain `research/` file (one hop down) — so you
  traverse *leaf → its bibliography*, never load the whole thing.
- New tool or technique? Add its canonical URL to the matching `research/<domain>.md` (keep the
  invariant: **every name in the tree resolves to a canonical URL here**), then cite it from the leaf.
  Policy: [research/meta-resources.md](research/meta-resources.md).

Back to [SKILL.md](SKILL.md) · the playbook tree: [references/00-map.md](references/00-map.md).
