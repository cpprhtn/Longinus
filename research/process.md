# ⛓️ process — methodology, severity & disclosure references

Bibliography for the governance & lens **spine**:
[`authorization-and-scope.md`](../references/authorization-and-scope.md) ·
[`attacker-mindset.md`](../references/attacker-mindset.md) ·
[`methodology.md`](../references/methodology.md) ·
[`audit-modes.md`](../references/audit-modes.md) ·
[`chaining-and-impact.md`](../references/chaining-and-impact.md) ·
[`severity-and-triage.md`](../references/severity-and-triage.md) ·
[`enforce-forward.md`](../references/enforce-forward.md) ·
[`reporting-and-disclosure.md`](../references/reporting-and-disclosure.md).
↑ back to the [RESEARCH hub](../RESEARCH.md).

## Methodology & process

- **PTES (Penetration Testing Execution Standard)** — pre-engagement → intel gathering → threat
  modeling → vuln analysis → exploitation → post-exploitation → reporting. →
  http://www.pentest-standard.org/
- **NIST SP 800-115** — technical guide to information security testing. →
  https://csrc.nist.gov/pubs/sp/800/115/final
- **OSSTMM**, **MITRE ATT&CK** (TTP coverage → https://attack.mitre.org/), **Cyber Kill Chain** (framing).
- **OWASP WSTG / MASTG** — per-domain testing checklists (web → [web.md](web.md); mobile → [mobile.md](mobile.md)).
- **PortSwigger / HackerOne / Bugcrowd** disclosure & methodology guidance for bounty workflows. →
  https://docs.hackerone.com/ · https://www.bugcrowd.com/

## Attacker mindset & chaining

Grounding for [`attacker-mindset.md`](../references/attacker-mindset.md) (the generative principles)
and [`chaining-and-impact.md`](../references/chaining-and-impact.md) (composing findings into impact):

- **Confused deputy** — Norm Hardy, *The Confused Deputy* (1988, ACM `10.1145/54289.871709`), the
  canonical framing for SSRF/CSRF/SSO/agentic authority confusion. →
  https://en.wikipedia.org/wiki/Confused_deputy_problem
- **Saltzer & Schroeder**, *The Protection of Information in Computer Systems* (1975) — least privilege,
  complete mediation, fail-safe defaults; the design principles attackers invert. →
  https://www.cs.virginia.edu/~evans/cs551/saltzer/
- **Parser / impedance differentials** — PortSwigger Research / **James Kettle** (HTTP desync, "Browser-
  Powered Desync", web cache attacks). → https://portswigger.net/research · and **Orange Tsai**, *A New
  Era of SSRF — Exploiting URL Parsers* (Black Hat 2017). → https://blog.orange.tw/
- **PortSwigger Top 10 Web Hacking Techniques** — the annual canon of new generative techniques. →
  https://portswigger.net/research/top-10-web-hacking-techniques
- **Chaining in practice** — disclosed bug-chain reports indexed by type:
  [reddelexc/hackerone-reports](https://github.com/reddelexc/hackerone-reports); CTF multi-stage chains
  in [ctf.md](ctf.md). The *lethal trifecta* framing for LLM agents (Simon Willison) lives in
  [ai-llm.md](ai-llm.md).

## Verification & enforcement standards

Grounding for [`enforce-forward.md`](../references/enforce-forward.md) (kill the class + gate the
regression) and the per-category enforcement templates:

- **OWASP ASVS** — application security *verification* standard (web/API). →
  https://owasp.org/www-project-application-security-verification-standard/
- **CIS Benchmarks** — hardening baselines for cloud/OS/k8s (policy-as-code maps to these). →
  https://www.cisecurity.org/cis-benchmarks
- **SLSA** (supply-chain integrity) + **NIST SSDF** (SP 800-218, secure software dev). →
  https://slsa.dev/ · https://csrc.nist.gov/Projects/ssdf
- **NIST SP 800-63** (digital identity) + **FAPI** (financial-grade API hardening). →
  https://pages.nist.gov/800-63-3/ · https://openid.net/wg/fapi/
- **NIST AI RMF** (AI risk management). → https://www.nist.gov/itl/ai-risk-management-framework
- **C/C++:** SEI CERT C/C++, MISRA, ISO 26262 — see
  [`secure-coding-standards.md`](../references/secure-coding-standards.md).
- **Enforcement tooling:** [Semgrep](https://semgrep.dev/) (SAST) ·
  [OPA](https://www.openpolicyagent.org/) / [Conftest](https://www.conftest.dev/) (policy-as-code) ·
  checkov/tfsec · gitleaks/detect-secrets · [pre-commit](https://pre-commit.com/) · modelscan.

## Severity & scoring

- **CVSS 4.0** (FIRST) — base/threat/environmental scoring. → https://www.first.org/cvss/
  ([calculator](https://www.first.org/cvss/calculator/4.0))
- **EPSS** — exploit-prediction scoring for prioritization. → https://www.first.org/epss/
- **CISA KEV** — Known Exploited Vulnerabilities catalog. → https://www.cisa.gov/known-exploited-vulnerabilities-catalog
- **SSVC** — Stakeholder-Specific Vulnerability Categorization (decision-tree triage). →
  https://www.cisa.gov/ssvc
- **NVD / CVE** — vulnerability records. → https://nvd.nist.gov/ · https://www.cve.org/
- **CWE** for weakness classification, **CAPEC** for attack patterns. → https://cwe.mitre.org/ ·
  https://capec.mitre.org/

## Vulnerability disclosure & reporting standards

- **RFC 9116 (`security.txt`)** — machine-readable disclosure contact. → https://www.rfc-editor.org/rfc/rfc9116
- **ISO/IEC 29147** (vulnerability disclosure) & **ISO/IEC 30111** (vulnerability handling). →
  https://www.iso.org/standard/72311.html · https://www.iso.org/standard/69725.html
- **CERT/CC Coordinated Vulnerability Disclosure guide.** → https://vuls.cert.org/confluence/display/CVD
- **disclose.io** — disclosure policy templates / safe-harbor language. → https://disclose.io/

## Legal basis (authorization gate)

- **US CFAA (18 U.S.C. §1030)** → https://www.law.cornell.edu/uscode/text/18/1030
- **UK Computer Misuse Act 1990** → https://www.legislation.gov.uk/ukpga/1990/18/contents
- **Korea 정보통신망법 (EN, KLRI)** → https://elaw.klri.re.kr/eng_service/lawView.do?hseq=61545&lang=ENG

## Reporting tooling

- [CVSS 4.0 calculator](https://www.first.org/cvss/calculator/4.0) · [EPSS](https://www.first.org/epss/) ·
  [pandoc](https://pandoc.org/) · [Dradis](https://dradisframework.com/) ·
  [Faraday](https://faradaysec.com/) · [PlexTrac](https://plextrac.com/)

## Related references

Applies to every branch. Domain bibliographies: [recon.md](recon.md) · [web.md](web.md) ·
[api.md](api.md) · [identity.md](identity.md) · [secrets-supply-chain.md](secrets-supply-chain.md) ·
[ai-llm.md](ai-llm.md) · [cloud-infra.md](cloud-infra.md) · [mobile.md](mobile.md) · [ctf.md](ctf.md).
Meta & living-doc policy: [meta-resources.md](meta-resources.md).
