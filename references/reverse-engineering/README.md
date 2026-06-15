# 🔬 reverse-engineering/ — static & dynamic RE

Understanding a program with no source: how it works, where it's vulnerable, what it hides. RE
underpins malware analysis, vuln research (feeds [../binary-exploitation/README.md](../binary-exploitation/README.md)),
crackmes, firmware/IoT, and license/DRM analysis. A core CTF category and a real job (malware analyst,
vuln researcher).

> ⛔ RE binaries you're allowed to (CTF, your own, licensed for analysis). For suspected malware, work
> in an **isolated VM with no production network**.
>
> **Scope & depth.** A map of the sub-field, not an exhaustive manual — RE is deep enough to be a career.
> Use this to triage and pick an approach; go deep with **[CTF Wiki (reverse)](https://ctf-wiki.mahaloz.re/reverse/introduction/)**,
> *Practical Malware Analysis* (Sikorski & Honig), the **[Ghidra](https://ghidra-sre.org/)/IDA** docs,
> and **[HackTricks](https://book.hacktricks.wiki/)** for technique detail.

## Tooling

- **Disassemblers/decompilers:** **Ghidra** (free, great decompiler), **IDA Pro/Free**, **Binary
  Ninja**, **radare2/rizin + Cutter** (GUI). Decompiler output (C-like) is your fastest path to
  understanding.
- **Debuggers:** `gdb`+pwndbg/GEF (Linux), `x64dbg`/WinDbg (Windows), **Frida** (cross-platform
  instrumentation/hooking), `ltrace`/`strace`.
- **Triage:** `file`, `strings`, `binwalk` (embedded files/firmware), `nm`/`objdump`/`readelf`,
  `floss` (deobfuscated strings), `xxd`. For .NET: **dnSpy/ILSpy**; Java: **jadx/procyon**; Python:
  `uncompyle6`/`pycdc` + `pyinstxtractor`; Go/Rust have specialized helpers.

## Static analysis workflow

1. **Triage:** `file` (arch/format/packed?), `strings`/`floss` (URLs, flags, error msgs, format
   strings, suspicious APIs), `binwalk` (embedded blobs), imports (`objdump -T`/PE imports) → what does
   it *touch* (network, crypto, files, registry)?
2. **Load into Ghidra/IDA:** find `main`/entry, follow the decompiler. Identify the program's logic,
   the **check** you care about (license, password, "win" condition), and the data flow.
3. **Recognize patterns:** crypto constants (AES S-box, SHA/MD5 init values — `findcrypt`), standard
   libc, compiler idioms, switch tables, vtables (C++).
4. **Annotate** (rename functions/vars, set types) as you understand — RE is iterative comprehension.

## Dynamic analysis workflow

When static is slow or the code is obfuscated/packed, run it (in a VM/sandbox):
- **Debug** to the interesting check; inspect registers/memory/arguments at the decision point;
  patch a branch or a comparison to pass (crackme staple).
- **Hook with Frida:** intercept functions, dump arguments/return values, bypass checks at runtime,
  log crypto inputs/keys.
- **Trace:** `ltrace`/`strace`/API monitor to see calls without reading every instruction.
- **Behavioral (malware):** isolated VM + network sim (INetSim/FakeNet), process/file/registry/network
  monitors (Procmon, Wireshark) to observe what it does.

## Unpacking & anti-analysis

- **Packers/obfuscation:** detect (`die`/Detect-It-Easy, high entropy, few imports, UPX header). UPX →
  `upx -d`; custom packers → dump from memory after the unpack stub runs (OEP), or Frida/`Scylla`-style
  dump + import reconstruction.
- **Anti-debug / anti-VM / anti-RE:** `IsDebuggerPresent`/`ptrace` self-attach, timing checks, VM
  artifacts, integrity self-checks, control-flow flattening (OLLVM), string/API obfuscation. Bypass by
  patching the check, hooking the API to lie, or stepping past it in the debugger. (These are
  defense-in-depth in real software — they slow analysis, they don't stop it.)

## CTF reverse workflow (typical)

1. `file` + `strings` (sometimes the flag is right there).
2. Decompile in Ghidra; find the function that validates input / prints "correct".
3. Understand the transform (often: input → some encoding/xor/crypto → compare to a constant).
4. Either **reverse the algorithm** to compute the right input, **patch** the check, or **brute** if
   the space is small. For VM-based/obfuscated challenges, reconstruct the bytecode/VM semantics.
5. Tools: `angr` (symbolic execution) can auto-solve many "find the input that reaches `win`"
   challenges; `z3` for constraint-heavy checks.

## Feeds vuln research

RE locates the bug; [../binary-exploitation/README.md](../binary-exploitation/README.md) weaponizes it.
For firmware/IoT: `binwalk` to extract the filesystem, then RE the binaries and analyze the web/network
services ([../web/](../web/README.md)). Pair with fuzzing (AFL++) to find new bugs at scale.

## Defensive note

If you ship native/compiled code: don't rely on obfuscation for secrets (it *will* be reversed — see
[../mobile/README.md](../mobile/README.md) and [../secrets-and-supply-chain/secret-detection.md](../secrets-and-supply-chain/secret-detection.md));
keep secrets and trust on the server. Auditing the C/C++ **source** (not the compiled binary)? See
[../secure-coding-standards.md](../secure-coding-standards.md) (CERT-C/CWE · MISRA).

## References

[Ghidra](https://ghidra-sre.org/) / [IDA](https://hex-rays.com/ida-pro/) / [Binary Ninja](https://binary.ninja/) /
[radare2](https://rada.re/) docs · [CTF Wiki (reverse)](https://ctf-wiki.mahaloz.re/reverse/introduction/) ·
Practical Malware Analysis (Sikorski & Honig) · [angr docs](https://docs.angr.io/) · [Azeria Labs (ARM)](https://azeria-labs.com/).
Tool URLs & full bibliography: [research/ctf.md](../../research/ctf.md). Back to [tree](../00-map.md).
