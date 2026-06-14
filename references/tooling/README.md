# 🧰 tooling/ — curated tool catalog

Best-in-class tools per branch. Pick the *right* tool for the phase, run it **only in scope**, and
**always review the output** — a tool gives you candidates, not findings (see
[../severity-and-triage.md](../severity-and-triage.md): prove it or park it). Prefer the tools you can
explain; a confirmed manual finding beats a wall of unreviewed scanner hits.

> ⛔ Many of these are dual-use. Use them only on targets you own or are authorized to test
> ([../authorization-and-scope.md](../authorization-and-scope.md)).

## Recon & discovery → [../recon/](../recon/README.md)
- **Subdomains/assets:** `subfinder`, `amass`, `assetfinder`, `crt.sh`, `dnsx`, `puredns`/`shuffledns`,
  `gotator`/`dnsgen` (permutations).
- **Liveness/fingerprint:** `httpx`, Wappalyzer, `naabu`/`masscan`/`nmap` (ports/services).
- **Content/URLs:** `katana`, `gau`, `waybackurls`, `ffuf`/`feroxbuster` (dir/file brute),
  `arjun`/`x8` (params), `git-dumper`.
- **OSINT:** `theHarvester`, Shodan/Censys/FOFA, `sherlock`/`maigret`, `exiftool`, Google/GitHub dorks.
- **Templated checks:** `nuclei` (CVEs/exposures/misconfig/takeovers — curate templates).

## Web & API → [../web/](../web/README.md), [../api/](../api/README.md)
- **Proxy/all-rounder:** **Burp Suite** (+ extensions: Autorize, Param Miner, Turbo Intruder,
  Collaborator, DOM Invader), **OWASP ZAP** (free), `mitmproxy`, `caido`.
- **Injection/specific:** `sqlmap` (SQLi), `commix` (command), `tplmap`/manual (SSTI), `dalfox`/XSS
  Hunter (XSS), `interactsh` (OOB/SSRF/blind), `nosqlmap`.
- **API/GraphQL:** Postman/`hoppscotch`, `graphw00f`+`clairvoyance`/`graphql-cop` (GraphQL), Swagger/
  OpenAPI importers, `kiterunner` (API route brute), `jwt_tool` (JWT attacks).
- **Fuzzing/PoC:** `ffuf`, Turbo Intruder (race conditions/single-packet), `nuclei` custom templates.

## Secrets & supply chain → [../secrets-and-supply-chain/](../secrets-and-supply-chain/README.md)
- **Secrets:** `gitleaks`, `trufflehog` (`--only-verified`), GitHub/GitLab secret scanning, `dive`
  (image layers).
- **SCA / deps:** `osv-scanner`, `trivy`, `npm audit`/`pip-audit`/`govulncheck`/`bundler-audit`,
  OWASP Dependency-Check, `grype`, Dependabot/Renovate.
- **SBOM / integrity:** `syft` (SBOM), `cosign`/Sigstore (signing), SLSA tooling.

## AI / LLM → [../ai-llm/](../ai-llm/README.md)
- **Red-team frameworks:** `garak` (LLM vuln scanner), `promptfoo` (eval/red-team), PyRIT, `deepteam`,
  Giskard. Map findings to OWASP LLM Top 10:2025 / MITRE ATLAS. Manual injection testing is still
  primary — these assist, they don't replace understanding the pipeline.

## Cloud & infra → [../cloud-and-infra/](../cloud-and-infra/README.md)
- **IaC/container static:** `checkov`, `trivy config`/`trivy image`, `tfsec`, `kics`, `dockle`,
  `grype`.
- **Cloud posture (read-only):** `prowler`, `ScoutSuite`, `cloudfox`, `steampipe`.
- **K8s:** `kubescape`, `kube-bench`, `kube-hunter`, `trivy k8s`.
- **Cloud exploitation (authorized):** `pacu` (AWS), CloudGoat/flAWS for practice.

## Mobile → [../mobile/](../mobile/README.md)
- `MobSF` (automated static+dynamic), `jadx`/`apktool` (Android), `class-dump`/Hopper (iOS),
  **Frida**/**Objection** (instrumentation, pinning/root-detection bypass), Burp + device CA.

## Binary / pwn → [../binary-exploitation/](../binary-exploitation/README.md)
- `pwntools`, `gdb`+**pwndbg**/**GEF**, `checksec`, `ROPgadget`/`ropper`, `one_gadget`, `pwninit`,
  `angr` (symbolic), AFL++/libFuzzer (fuzzing), ASan/sanitizers.

## Reverse engineering → [../reverse-engineering/](../reverse-engineering/README.md)
- **Ghidra** (free decompiler), IDA, Binary Ninja, `radare2`/`rizin`+Cutter, `binwalk`, `floss`,
  `strings`, Frida, `dnSpy`/ILSpy (.NET), `jadx` (Java), `x64dbg`/WinDbg (Windows), `angr`/`z3`.

## Cryptography → [../cryptography/](../cryptography/README.md)
- `pycryptodome`, **SageMath**, `sympy`/`z3`, `RsaCtfTool`, `hashcat`/`john` (cracking), `hashpump`
  (length-extension), `factordb`/`yafu`, `jwt_tool`, `featherduster`.

## Forensics → [../forensics/](../forensics/README.md)
- **Volatility 3** (memory), **Wireshark**/`tshark`/NetworkMiner (pcap), **Autopsy**/Sleuth Kit,
  `binwalk`/`foremost`/`scalpel` (carving), `exiftool`, `zsteg`/`steghide`/`stegsolve` (stego),
  `testdisk`/`photorec` (recovery), Audacity/Sonic Visualiser (audio).

## Reporting & scoring → [../reporting-and-disclosure.md](../reporting-and-disclosure.md)
- CVSS 4.0 calculator (FIRST), EPSS lookups, CWE/CAPEC, Markdown/`pandoc` for reports, Dradis/Faraday/
  PlexTrac (engagement management).

## All-in-one platforms & learning
- **Distros:** Kali Linux, Parrot OS, BlackArch (the tools above, pre-packaged).
- **Practice (sanctioned):** PortSwigger Web Security Academy, HackTheBox, TryHackMe, pwn.college,
  CryptoHack, CTFtime events, OWASP Juice Shop, DVWA.

## Meta-tools / references
- **PayloadsAllTheThings** (payload encyclopedia), **HackTricks** (technique wiki), **SecLists**
  (wordlists), GTFOBins/LOLBAS (living-off-the-land), exploit-db/searchsploit.

All canonical URLs in [../../RESEARCH.md](../../RESEARCH.md). Back to [tree](../00-map.md).
