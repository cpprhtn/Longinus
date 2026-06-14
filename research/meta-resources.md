# 🧰 meta-resources — wordlists, practice ranges & living-document policy

Cross-cutting resources used across every branch, plus the policy for keeping this bibliography
current. ↑ back to the [RESEARCH hub](../RESEARCH.md).

## Payload / technique encyclopedias

- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) ·
  [HackTricks](https://book.hacktricks.wiki/) · [GTFOBins](https://gtfobins.github.io/) ·
  [LOLBAS](https://lolbas-project.github.io/) · [Exploit-DB / searchsploit](https://www.exploit-db.com/)

## Curated "awesome" lists (further reading)

General hubs (domain-specific lists are linked from each `research/<domain>.md`):

- [enaqx/awesome-pentest](https://github.com/enaqx/awesome-pentest) — pentest tools & resources.
- [Hack-with-Github/Awesome-Hacking](https://github.com/Hack-with-Github/Awesome-Hacking) — meta-index
  of awesome lists for hackers/pentesters/researchers.
- Per domain: web → [web.md](web.md) · bug-bounty/recon → [recon.md](recon.md) · AI/LLM →
  [ai-llm.md](ai-llm.md) · cloud → [cloud-infra.md](cloud-infra.md) · mobile → [mobile.md](mobile.md) ·
  CTF (pwn/crypto/forensics) → [ctf.md](ctf.md).

## Recon / OSINT meta

- [OSINT Framework](https://osintframework.com/) (tool index) · bug-bounty recon methodology
  references (PortSwigger / HackerOne / Bugcrowd — see [process.md](process.md)). Recon tools: [recon.md](recon.md).

## Real-world cases & writeup sources

Where to find the actual code/PoCs behind disclosed bugs and CTF solves:

- **Bug-bounty disclosures:** [HackerOne Hacktivity](https://hackerone.com/hacktivity) ·
  [Pentester Land writeups](https://pentester.land/writeups/) (curated feed) ·
  [reddelexc/hackerone-reports](https://github.com/reddelexc/hackerone-reports) — top disclosed H1
  reports **indexed by bug type** (each playbook leaf links its matching `TOP<TYPE>.md` page).
- **Original research / advisories:** [PortSwigger Research](https://portswigger.net/research)
  (request smuggling, web cache poisoning, SSRF…) · [Google Project Zero](https://googleprojectzero.blogspot.com/)
  · [GitHub Security Lab](https://securitylab.github.com/) · [ZDI / Pwn2Own blog](https://www.zerodayinitiative.com/blog).
- **CTF writeups:** [CTFtime writeups](https://ctftime.org/writeups) (per-event, per-challenge) — plus
  the pwn/RE/crypto/forensics walkthrough hubs in [ctf.md](ctf.md).
- **Landmark cases** are cited in the matching playbook leaf's "Real-world cases" footer: SSRF→Capital
  One; supply chain→Dependency Confusion / XZ backdoor; deserialization→FoxGlove; injection→Log4Shell;
  identity→JWT `alg` flaws; prompt injection→Embrace The Red.

## Wordlists

- [SecLists](https://github.com/danielmiessler/SecLists)

## Distros

- [Kali Linux](https://www.kali.org/) · [Parrot OS](https://www.parrotsec.org/) ·
  [BlackArch](https://blackarch.org/) (the branch tools, pre-packaged).

## Sanctioned practice ranges

- [PortSwigger Academy](https://portswigger.net/web-security/all-labs) ·
  [HackTheBox](https://www.hackthebox.com/) · [TryHackMe](https://tryhackme.com/) ·
  [pwn.college](https://pwn.college/) · [CryptoHack](https://cryptohack.org/) ·
  [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) · [DVWA](https://github.com/digininja/DVWA) ·
  [CTFtime](https://ctftime.org/)

## Related projects & prior art (other Claude Code security skills)

Other community efforts that package offensive-security workflows as Claude Code skills/agents — useful
for comparison and cross-pollination (Longinus stays a single traversable tree with a hard auth gate
and "prove it or park it"):

- [Eyadkelleh/awesome-skills-security](https://github.com/Eyadkelleh/awesome-skills-security) — SecLists
  wordlists, injection payloads & expert agents packaged as skills.
- [mahmutka/cybersecurity-claude-skills](https://github.com/mahmutka/cybersecurity-claude-skills) — web
  hacking, pentest recon, secure-code-review, CTF-solver skills.
- [Orizon-eu/claude-code-pentest](https://github.com/Orizon-eu/claude-code-pentest) — 6 skills covering
  the recon→exploit-chain→report lifecycle.
- [frendysanusi/claude-pentest-skills](https://github.com/frendysanusi/claude-pentest-skills) —
  scope-enforcing, OWASP-aligned skill pack with a 6-gate validation flow.
- [transilienceai/communitytools](https://github.com/transilienceai/communitytools) — skills/agents +
  a Kali-in-a-container runner.
- [Stickman230/claude-pentest](https://github.com/Stickman230/claude-pentest) — agents + Kali MCP
  integration.

> ⚠️ These are third-party; vet their authorization model and safety posture before use. Same gate
> applies: [../references/authorization-and-scope.md](../references/authorization-and-scope.md).

## Living-document policy

These references drift (versions, URLs, ownership). Before relying on a specific control or version,
confirm against the canonical URL. When you extend a leaf doc with a new technique or name a new tool,
add its source to the matching domain file under `research/` so the bibliography stays complete — every
leaf points back to its domain `research/<branch>.md` (and the [hub](../RESEARCH.md)) for the canonical link.

## Related references

[RESEARCH hub](../RESEARCH.md) · [rationale.md](rationale.md) · [process.md](process.md) · all domain files.
