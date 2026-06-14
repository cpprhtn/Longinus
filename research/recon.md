# 🔭 recon — references & tooling

Bibliography for the [`references/recon/`](../references/recon/README.md) branch (attack-surface
discovery, OSINT). ↑ back to the [RESEARCH hub](../RESEARCH.md).

## Frameworks & standards

- **OWASP WSTG — Information Gathering** (WSTG-INFO): the canonical surface-mapping checklist. →
  https://owasp.org/www-project-web-security-testing-guide/
- **PTES — Intelligence Gathering**: the recon stage of the pentest standard. → http://www.pentest-standard.org/
- **NIST SP 800-115** — technical testing (incl. discovery/scanning). → https://csrc.nist.gov/pubs/sp/800/115/final
- **OSINT Framework** — index of OSINT tools/sources. → https://osintframework.com/

## Tooling

> Passive recon (public data only) is generally safe; **active** probing needs authorization — see
> [process.md](process.md). Dual-use: run only in scope.

- Subdomains/assets: [subfinder](https://github.com/projectdiscovery/subfinder) ·
  [amass](https://github.com/owasp-amass/amass) · [assetfinder](https://github.com/tomnomnom/assetfinder) ·
  [crt.sh](https://crt.sh/) · [dnsx](https://github.com/projectdiscovery/dnsx) ·
  [puredns](https://github.com/d3mondev/puredns) · [shuffledns](https://github.com/projectdiscovery/shuffledns) ·
  [dnsgen](https://github.com/ProjectAnte/dnsgen) · [gotator](https://github.com/Josue87/gotator) ·
  [altdns](https://github.com/infosec-au/altdns)
- Liveness/fingerprint: [httpx](https://github.com/projectdiscovery/httpx) ·
  [Wappalyzer](https://www.wappalyzer.com/) · [naabu](https://github.com/projectdiscovery/naabu) ·
  [masscan](https://github.com/robertdavidgraham/masscan) · [nmap](https://nmap.org/)
- Content/URLs/params: [katana](https://github.com/projectdiscovery/katana) ·
  [gau](https://github.com/lc/gau) · [waybackurls](https://github.com/tomnomnom/waybackurls) ·
  [ffuf](https://github.com/ffuf/ffuf) · [feroxbuster](https://github.com/epi052/feroxbuster) ·
  [arjun](https://github.com/s0md3v/Arjun) · [x8](https://github.com/Sh1Yo/x8) ·
  [uro](https://github.com/s0md3v/uro)/[qsreplace](https://github.com/tomnomnom/qsreplace) (URL/param dedupe) ·
  [LinkFinder](https://github.com/GerbenJavado/LinkFinder)/[xnLinkFinder](https://github.com/xnl-h4ck3r/xnLinkFinder) (endpoints in JS) ·
  [git-dumper](https://github.com/arthaud/git-dumper)
- OSINT: [theHarvester](https://github.com/laramies/theHarvester) · [Shodan](https://www.shodan.io/) ·
  [Censys](https://search.censys.io/) · [FOFA](https://en.fofa.info/) · [ZoomEye](https://www.zoomeye.ai/) ·
  [sherlock](https://github.com/sherlock-project/sherlock) · [maigret](https://github.com/soxoj/maigret) ·
  [SecurityTrails](https://securitytrails.com/) · [grayhatwarfare](https://buckets.grayhatwarfare.com/) ·
  [Have I Been Pwned](https://haveibeenpwned.com/) · [Hunter.io](https://hunter.io/) ·
  [BuiltWith](https://builtwith.com/) · [bgp.he.net](https://bgp.he.net/) (ASN/IP) ·
  doc metadata: [metagoofil](https://github.com/laramies/metagoofil) · [FOCA](https://github.com/ElevenPaths/FOCA)
- Templated checks: [nuclei](https://github.com/projectdiscovery/nuclei) ·
  takeover: [subjack](https://github.com/haccer/subjack)

## Further reading (curated lists)

- [vavkamil/awesome-bugbounty-tools](https://github.com/vavkamil/awesome-bugbounty-tools) — broad
  bug-bounty tool index.
- [amrelsagaei/Bug-Bounty-Hunting-Methodology-2025](https://github.com/amrelsagaei/Bug-Bounty-Hunting-Methodology-2025)
  — recon→enumeration→testing methodology (2025 edition).
- General hubs: [enaqx/awesome-pentest](https://github.com/enaqx/awesome-pentest) ·
  [Hack-with-Github/Awesome-Hacking](https://github.com/Hack-with-Github/Awesome-Hacking).

## Related references

Feeds the testing branches: [web.md](web.md) · [api.md](api.md). Leaked-secret discovery →
[secrets-supply-chain.md](secrets-supply-chain.md). Engagement flow & authorization →
[process.md](process.md). Practice ranges/wordlists → [meta-resources.md](meta-resources.md).
