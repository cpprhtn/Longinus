# 💡 rationale — design idea, AI-code risk evidence & CTF taxonomy

The "why" behind Longinus. ↑ back to the [RESEARCH hub](../RESEARCH.md).

## 1. Design rationale (the idea, organized)

**Problem.** Security is an afterthought for most builders, and AI-assisted ("vibe") coding has made
that worse: code ships faster than anyone can review it, and the generated code is *measurably* less
secure. Meanwhile the people who *can* find these bugs — bug-bounty hunters, pentesters, CTF players —
work from structured methodologies and deep sub-specialties that an ordinary developer never learns.

**Thesis.** Encode that offensive methodology as a skill so any developer can run it (or have an AI
run it) against their own work, *and* so authorized testers get a repeatable, deep playbook. Offense
in service of defense.

**Three pillars** (map to the three use-cases):

| Pillar | Audience | Primary branches |
|---|---|---|
| Defensive self-audit | solo devs, founders, vibe coders | `secrets-and-supply-chain`, `web`, `identity`, `ai-llm`, `cloud-and-infra` |
| Authorized offense | bounty hunters, pentesters | `recon`, `web`, `api`, `identity` + methodology/reporting spine |
| Specialist depth | CTF players, security learners | `binary-exploitation`, `reverse-engineering`, `cryptography`, `forensics`, `mobile` |

**Design constraints derived from the research:**

- *Low false positives.* AI scanners are noisy; the value is in confirmed, reproducible findings. The
  methodology enforces "prove it or park it."
- *Authorization first.* The single biggest risk of an offensive tool is misuse. A hard gate precedes
  any active probing; static code audit is always available because you own your code.
- *Fix-forward.* Every finding ends in a remediation, ideally a patch.
- *Tree shape.* Professional offense is organized by sub-field; the docs mirror that so users can go
  exactly as deep as needed. Each leaf is self-contained and read on demand — the tree is *traversed*,
  not read front-to-back.

## 2. Vibe-coding / AI-generated-code risk evidence

The numbers that justify the "self-audit" pillar:

- ~**45%** of AI-generated code samples introduced a known security weakness in controlled tests;
  AI code shows ~**2.74×** the vulnerability density of human-written code (Veracode 2025 GenAI Code
  Security Report; academic studies). → https://www.veracode.com/resources/analyst-reports/2025-genai-code-security-report/
- Y Combinator W25: ~**25%** of startups reported codebases ~95% AI-generated; researchers scanning
  ~5,600 vibe-coded apps found 2,000+ vulnerabilities and 400+ exposed secrets. (Cloud Security
  Alliance / industry reporting.) → https://cloudsecurityalliance.org/blog/2025/07/09/understanding-security-risks-in-ai-generated-code
- Dominant failure patterns: **hallucinated/typosquatted dependencies (slopsquatting)**, **outdated
  libraries with known CVEs**, **hard-coded secrets**, **missing/broken authorization**, **prompt
  injection** in LLM-backed features. (Endor Labs, CSA, Help Net Security 2025.) →
  https://www.endorlabs.com/learn/the-most-common-security-vulnerabilities-in-ai-generated-code
- **Slopsquatting** original research (hallucinated-package reproducibility): "We Have a Package for
  You" — Spracklen et al., 2024/25. → https://arxiv.org/abs/2406.10279

(Maps to [secrets-supply-chain.md](secrets-supply-chain.md) and [ai-llm.md](ai-llm.md).)

## 3. CTF / specialist taxonomy

Standard CTF (Jeopardy) categories, which map to professional sub-fields and to the specialist
branches of this tree:

| CTF category | Tree branch | Real-world role |
|---|---|---|
| Web exploitation | `web/`, `api/` | AppSec / web pentester |
| Binary exploitation (pwn) | `binary-exploitation/` | Exploit dev / vuln research |
| Reverse engineering | `reverse-engineering/` | Malware analyst / RE |
| Cryptography | `cryptography/` | Crypto engineer / appsec |
| Forensics | `forensics/` | DFIR / incident response |
| OSINT / recon | `recon/` | Recon / threat intel |
| Misc / hardware / Web3 / AI-ML | `ai-llm/`, (hardware/blockchain noted in `references/00-map.md`) | emerging specialties |

Sources: CTF Wiki, HackerDNA CTF category overview, Snyk CTF types, CTFtime. →
https://ctf-wiki.mahaloz.re/introduction/content/ · https://ctftime.org/

(Tooling for the specialist branches → [ctf.md](ctf.md); practice ranges → [meta-resources.md](meta-resources.md).)

## Related references

[RESEARCH hub](../RESEARCH.md) · [process.md](process.md) · every domain file linked from the hub.
