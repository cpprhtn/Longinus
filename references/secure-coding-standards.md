# 🧷 Secure-coding standards — C/C++, embedded & safety-critical source

For a **source audit of native code** (C/C++, firmware, embedded, kernel, automotive/medical/avionics),
the question isn't just "is there an exploitable bug" — it's "does this code comply with the
secure-/safe-coding standard the project is held to." This leaf is the source-compliance companion to
the *exploitation* branches ([binary-exploitation/](binary-exploitation/README.md),
[reverse-engineering/](reverse-engineering/README.md)): those weaponize a compiled binary; this audits
the C/C++ **source** against the rules that prevent the bug in the first place.

## When this applies (routing trigger)

Open this leaf when the target is **C/C++/native source** — a `*.c`/`*.cpp`/`*.h`, a `CMakeLists.txt`/
`Makefile`, firmware/RTOS code, a kernel module, or anything in a **safety-critical** regime
(automotive/ISO 26262, medical/IEC 62304, avionics/DO-178C, industrial/IEC 61508). For pure
memory-safe stacks (Rust/Go/managed languages) this is mostly moot — say so honestly.

## The standards landscape (pick by goal — they are NOT interchangeable)

| Standard | Primary goal | Use it when |
|---|---|---|
| **SEI CERT C / C++** | **Security** — rules that prevent exploitable UB (the security-primary choice) | any security audit of C/C++ → start here, alongside CWE |
| **CWE** | Cross-cutting weakness taxonomy (you already triage with it) | classify every finding ([severity-and-triage.md](severity-and-triage.md)) |
| **MISRA C / C++** | **Safety, reliability, portability** — avoid undefined/unspecified behavior in safety-critical embedded | automotive/embedded *compliance* mandates; safety-critical builds |
| **ISO 26262 / IEC 61508 / DO-178C / IEC 62304** | Functional-safety *process* regimes (often **require** MISRA) | the project is in that domain and must certify |
| **BARR-C, JPL, AUTOSAR C++14** | House/embedded style + safety rules (largely MISRA-compatible) | the team has adopted one of these |

> **The honest distinction (state it in the report):** **MISRA's purpose is safety/reliability, not
> security** — many of its rules happen to prevent the same UB that causes memory-corruption bugs, but
> for a *security* audit **CERT C/C++ + CWE are the on-point standards**, and MISRA is the *companion*
> you add when the project is also safety-critical / contractually MISRA-bound. Don't report "MISRA
> violation" as if it were a vulnerability; report the security impact (CERT/CWE) and note the MISRA
> rule it also breaks.

## Where the bugs hide (the high-yield native constructs)

- **Unbounded copies:** `gets`, `strcpy`/`strcat`, `sprintf`, `scanf("%s")`, `memcpy`/`memmove` with an
  attacker-influenced length → buffer overflow (CWE-787/120).
- **Integer issues:** size/length arithmetic that can overflow/underflow or signed↔unsigned confusion
  before a `malloc`/`memcpy` (CWE-190/191) → undersized allocation → heap overflow.
- **Format strings:** `printf(user)`, `syslog(user)` (CWE-134).
- **Memory lifecycle:** use-after-free, double-free, missing `free`, returning a stack address
  (CWE-416/415/401).
- **Unchecked returns:** `malloc`/`realloc`/`fopen` result used without a NULL check (CWE-252/690).
- **Off-by-one / missing NUL terminator**, `strncpy` that doesn't terminate, array index from input.

## Mechanical scan

> Run these greps, apply SKIP, report with the fixed severity (provisional — confirm reachability before
> acting; see [audit-modes.md](audit-modes.md)). Map each to CERT-C + CWE; add the MISRA rule if the
> project is MISRA-bound.

**STEP 1 — Unbounded string/memory ops** → CWE-120/787 · CERT `STR31-C`/`ARR38-C`
```bash
rg -n "\bgets\s*\(|\bstrcpy\s*\(|\bstrcat\s*\(|\bsprintf\s*\(|\bscanf\s*\([^)]*%s|\bmemcpy\s*\(|\balloca\s*\(" .
```
- **SKIP:** length is a compile-time constant ≤ destination size; uses the `_s`/`strlcpy`/`snprintf` bounded variant
- **FINDING:** Buffer overflow · **High** (Critical if the length is attacker-controlled) · Fix: bounded API + explicit size

**STEP 2 — Format string** → CWE-134 · CERT `FIO30-C`
```bash
rg -n "printf\s*\(\s*[a-zA-Z_][a-zA-Z0-9_]*\s*\)|syslog\s*\([^,]*,\s*[a-zA-Z_][a-zA-Z0-9_]*\s*\)|fprintf\s*\([^,]*,\s*[a-zA-Z_]" .
```
- **SKIP:** the first/format arg is a string literal
- **FINDING:** Format string · **High** · Fix: `printf("%s", user)` — never a user-controlled format

**STEP 3 — Integer overflow into allocation/copy** → CWE-190 · CERT `INT30-C`/`MEM35-C`
```bash
rg -n "malloc\s*\([^)]*[*+][^)]*\)|memcpy\s*\([^,]*,[^,]*,[^)]*[*+][^)]*\)" .
```
- **SKIP:** operands are bounds-checked before the arithmetic; uses an overflow-checked allocator
- **FINDING:** Integer overflow → heap overflow · **High** · Fix: validate sizes; checked arithmetic

**STEP 4 — Unchecked allocation / memory lifecycle** → CWE-252/416 · CERT `MEM52-C`/`EXP34-C`
```bash
rg -n "free\s*\(" .          # inspect for double-free / use-after-free around each
rg -n "= *malloc\s*\(|= *realloc\s*\(" .   # confirm each result is NULL-checked before use
```
- **SKIP:** the pointer is NULL-checked before dereference; freed pointer is set to NULL and not reused
- **FINDING:** UAF/double-free **Critical** · NULL-deref **Medium** · Fix: check returns; null-after-free

**Output template (quick mode):**
```
| File:Line | Type | Severity | CERT-C / CWE (+ MISRA) | Fix |
|---|---|---|---|---|
```

## How to find it (tooling)

- **Static / linters:** **cppcheck** (free; has a **MISRA** add-on and CERT mapping), **clang-tidy**
  (`cert-*`, `bugprone-*`, `cppcoreguidelines-*` checks), **Clang Static Analyzer**, **Coverity**,
  **Polyspace**, **PC-lint Plus**, **Frama-C** (formal). Run with project compile flags.
- **Compiler as a linter:** `-Wall -Wextra -Wconversion -Wformat=2 -Werror`; `-fanalyzer` (GCC).
- **Dynamic (confirm reachability):** **ASan/UBSan/MSan**, **Valgrind**, and **fuzzing** (libFuzzer/
  AFL++) — the fastest way to turn a "possible" into a proven crash → hand to
  [binary-exploitation/](binary-exploitation/README.md).

## How to confirm & fix

- **Confirm:** the tainted source→sink path in the C source, ideally a sanitizer/fuzzer crash that
  proves it's reachable with attacker input. A linter hit alone is "needs validation."
- **Fix-forward:** replace the unsafe API with its bounded form (`snprintf`, `strlcpy`, `memcpy_s`);
  validate sizes with checked arithmetic; check every allocation; **adopt CERT-C/MISRA as a CI linter
  gate** (cppcheck/clang-tidy failing the build); enable warnings-as-errors + sanitizers in CI; and
  where the domain allows, **prefer a memory-safe language** (Rust) for new code.

## CTF angle

Source-review challenges hand you C and ask you to spot the overflow/UAF/format-string before writing
the pwn exploit — the patterns above are the checklist; then pivot to
[binary-exploitation/](binary-exploitation/README.md) to weaponize it.

## References

[SEI CERT C](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard) ·
[SEI CERT C++](https://wiki.sei.cmu.edu/confluence/display/cplusplus/SEI+CERT+C%2B%2B+Coding+Standard) ·
[MISRA](https://misra.org.uk/) · [ISO 26262](https://www.iso.org/standard/68383.html) ·
[BARR-C](https://barrgroup.com/embedded-systems/books/embedded-c-coding-standard) ·
[CWE](https://cwe.mitre.org/) · tools: [cppcheck](https://cppcheck.sourceforge.io/) ·
[clang-tidy](https://clang.llvm.org/extra/clang-tidy/). Triage with
[severity-and-triage.md](severity-and-triage.md); exploit the compiled artifact via
[binary-exploitation/](binary-exploitation/README.md). Full bibliography:
[research/process.md](../research/process.md) (secure-coding standards). Back to [tree](00-map.md).
