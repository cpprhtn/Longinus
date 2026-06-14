# 🧩 ctf — references & tooling (pwn · reverse · crypto · forensics)

Bibliography for the four CTF-specialist branches:
[`binary-exploitation/`](../references/binary-exploitation/README.md) ·
[`reverse-engineering/`](../references/reverse-engineering/README.md) ·
[`cryptography/`](../references/cryptography/README.md) ·
[`forensics/`](../references/forensics/README.md). ↑ back to the [RESEARCH hub](../RESEARCH.md).

> Practice on CTF challenges, your own binaries, and sanctioned labs — see [meta-resources.md](meta-resources.md)
> for practice ranges and [process.md](process.md) for the authorization gate.

## Frameworks, curricula & references

- **CTF Wiki (EN)** — pwn, reverse, crypto, forensics, misc. → https://ctf-wiki.mahaloz.re/
  (sections: [pwn](https://ctf-wiki.mahaloz.re/pwn/readme/) ·
  [reverse](https://ctf-wiki.mahaloz.re/reverse/introduction/) ·
  [crypto](https://ctf-wiki.mahaloz.re/crypto/introduction/) ·
  [misc/forensics](https://ctf-wiki.mahaloz.re/misc/introduction/))
- **CTF Field Guide (Trail of Bits).** → https://trailofbits.github.io/ctf/
- **pwn.college** — structured binary-exploitation curriculum. → https://pwn.college/
- **ROP Emporium** — ROP-focused challenge set. → https://ropemporium.com/
- **Nightmare** (guyinatuxedo) — pwn walkthroughs. → https://guyinatuxedo.github.io/
- **how2heap** (shellphish) — glibc heap technique reference. → https://github.com/shellphish/how2heap
- **pwnable.kr / pwnable.tw** — pwn challenge sites. → http://pwnable.kr/ · https://pwnable.tw/
- **Azeria Labs** — ARM assembly & exploitation. → https://azeria-labs.com/
- **Practical Malware Analysis** (Sikorski & Honig) — the standard RE/malware text.
- **CryptoHack** — applied-crypto challenges. → https://cryptohack.org/
- **Cryptopals** — crypto attack set. → https://cryptopals.com/
- **OWASP A04:2025 Cryptographic Failures** + [Cryptographic Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html)
  — real-world crypto misuse (also see [web.md](web.md)).
- **SANS DFIR** posters & cheat sheets. → https://www.sans.org/posters/?focus-area=digital-forensics-incident-response
- **MITRE ATT&CK** (DFIR mapping). → https://attack.mitre.org/

## Tooling

### Binary / pwn
- [pwntools](https://github.com/Gallopsled/pwntools) ([docs](https://docs.pwntools.com/)) ·
  [pwndbg](https://github.com/pwndbg/pwndbg) · [GEF](https://github.com/hugsy/gef) ·
  [checksec](https://github.com/slimm609/checksec.sh) ·
  [ROPgadget](https://github.com/JonathanSalwan/ROPgadget) · [ropper](https://github.com/sashs/Ropper) ·
  [one_gadget](https://github.com/david942j/one_gadget) · [pwninit](https://github.com/io12/pwninit) ·
  [angr](https://angr.io/) · [AFL++](https://github.com/AFLplusplus/AFLplusplus) ·
  [libFuzzer](https://llvm.org/docs/LibFuzzer.html)

### Reverse engineering
- [Ghidra](https://ghidra-sre.org/) ([repo](https://github.com/NationalSecurityAgency/ghidra)) ·
  [IDA](https://hex-rays.com/ida-pro/) · [Binary Ninja](https://binary.ninja/) ·
  [radare2](https://rada.re/) / [rizin](https://rizin.re/) + [Cutter](https://cutter.re/) ·
  [binwalk](https://github.com/ReFirmLabs/binwalk) · [FLOSS](https://github.com/mandiant/flare-floss) ·
  [dnSpy](https://github.com/dnSpy/dnSpy) · [ILSpy](https://github.com/icsharpcode/ILSpy) (.NET) ·
  [x64dbg](https://x64dbg.com/) · [WinDbg](https://learn.microsoft.com/windows-hardware/drivers/debugger/) ·
  [Hopper](https://www.hopperapp.com/) (macOS/iOS) ·
  [Detect-It-Easy](https://github.com/horsicq/Detect-It-Easy) · [z3](https://github.com/Z3Prover/z3) ·
  Python bytecode: [uncompyle6](https://github.com/rocky/python-uncompyle6)/[pycdc](https://github.com/zrax/pycdc) +
  [pyinstxtractor](https://github.com/extremecoders-re/pyinstxtractor) · import rebuild: [Scylla](https://github.com/NtQuery/Scylla) ·
  malware sandbox/sim: [INetSim](https://www.inetsim.org/) · [FakeNet-NG](https://github.com/mandiant/flare-fakenet-ng) ·
  [Procmon (Sysinternals)](https://learn.microsoft.com/sysinternals/downloads/procmon)

### Cryptography
- [PyCryptodome](https://www.pycryptodome.org/) · [SageMath](https://www.sagemath.org/) ·
  [SymPy](https://www.sympy.org/) · [RsaCtfTool](https://github.com/RsaCtfTool/RsaCtfTool) ·
  [hashcat](https://hashcat.net/hashcat/) · [John the Ripper](https://www.openwall.com/john/) ·
  [HashPump](https://github.com/bwall/HashPump) · [factordb](http://factordb.com/) ·
  [yafu](https://github.com/bbuhrow/yafu) · [FeatherDuster](https://github.com/nccgroup/featherduster)

### Forensics
- [Volatility 3](https://github.com/volatilityfoundation/volatility3)
  ([foundation](https://www.volatilityfoundation.org/)) · [Wireshark](https://www.wireshark.org/) ·
  [NetworkMiner](https://www.netresec.com/?page=NetworkMiner) · [Autopsy](https://www.autopsy.com/) /
  [Sleuth Kit](https://www.sleuthkit.org/) · [foremost](http://foremost.sourceforge.net/) ·
  [scalpel](https://github.com/sleuthkit/scalpel) · [ExifTool](https://exiftool.org/) ·
  [zsteg](https://github.com/zed-0xff/zsteg) · [steghide](https://steghide.sourceforge.net/) ·
  [StegCracker](https://github.com/Paradoxis/StegCracker) · [Stegsolve](http://www.caesum.com/handbook/stego.html) ·
  [TestDisk / PhotoRec](https://www.cgsecurity.org/) · [Audacity](https://www.audacityteam.org/) ·
  [Sonic Visualiser](https://www.sonicvisualiser.org/) (spectrogram) ·
  PDF: [peepdf](https://github.com/jesparza/peepdf) / [pdf-tools (Didier Stevens)](https://blog.didierstevens.com/programs/pdf-tools/)

## Further reading (curated lists)

- [apsdehal/awesome-ctf](https://github.com/apsdehal/awesome-ctf) — CTF frameworks, tools & wargames
  across every category.
- [sobolevn/awesome-cryptography](https://github.com/sobolevn/awesome-cryptography) — crypto libraries,
  courses & resources.
- [cugu/awesome-forensics](https://github.com/cugu/awesome-forensics) — DFIR tools & references.
- **Team writeups & challenge archives (real solve code):**
  [perfectblue/ctf-writeups](https://github.com/perfectblue/ctf-writeups) ·
  [p4-team/ctf](https://github.com/p4-team/ctf) ·
  [balsn/ctf_writeup](https://github.com/balsn/ctf_writeup) ·
  [sajjadium/ctf-archives](https://github.com/sajjadium/ctf-archives) (challenge files) ·
  per-event solves on [CTFtime writeups](https://ctftime.org/writeups).

## Related references

[identity.md](identity.md) (JWT `alg` / HMAC cracking) · [web.md](web.md) (web-CTF, TLS/crypto misuse) ·
[mobile.md](mobile.md) (native-lib RE) · [rationale.md](rationale.md) (CTF→role taxonomy) ·
[meta-resources.md](meta-resources.md) (practice ranges) · [process.md](process.md).
