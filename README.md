# RochedDc

> Advanced Python obfuscation and native hardening toolkit `discord.RochedDc.cc | RochedDc.cc`

## What it does

RochedDc transforms Python source into a hardened, tamper-resistant artifact. The output is not just obfuscated it is actively hostile to reverse engineering, dynamic analysis, and tampering.

## Features

### Kernel & Lowering
- **141 native operations** Python builtins replaced with C++ kernel implementations across 14 families
- **Lowered families (on by default):** I/O, scalar, string, container, service, protocol, function, import, fusion
- **Opt-in families:** arithmetic, truth, slice, format, augmented-assignment
- **Nested call fusion** consecutive lowered calls collapsed into a single kernel dispatch; dedicated fast paths for fusion and f-strings
- **Performance:** `0.19x–1.56x` of plain CPython 3.12 across lowered families (import `0.37x`, container `0.86x–0.94x`, scalar `0.81x–0.82x`)

### Tamper Protection
- **Compiled kernel exports randomized per build** no stable export names across builds
- **Runtime name resolution** kernel operation names resolved per-process, never emitted as literals
- **Split-secret key derivation** bootstrap derives key material from a fragmented decoder scheme, no single embedded key
- **Sealed sections** payload split into individually sealed sections, streamed one at a time; the full code object is never resident in a single readable buffer
- **Guard pages** sealed sections can be placed on guard pages that trap any out-of-window access
- **Memory protection** key and white-box pages pinned; sections re-protected after load (no longer writable)
- **Single failure path** any tamper trigger wipes key material and exits on first strike (exit 137)
- **Self-corruption on tamper** runs from a detached child process; waits for parent exit, then overwrites artifact with same-length random garbage (opt-out available); replaces previous self-delete behavior
- **Byte-tamper sweep** 12 single-byte mutations across two randomized builds, every run fail-closed, 8–9 hitting the kernel kill ladder

### Anti-Analysis
- **Native anti-debug tier** debug register, debug port, and toolchain checks inside the kernel
- **Audit-hook detection** timing detection, hook-addition detection, detection of hooks planted from `site-packages`
- **Timing side-channel detector** based on clock-interrupt bias
- **Watchdog thread** heartbeat liveness pairing with a timeout kill
- **Crash-dialog suppression** native kill exits cleanly, no error prompt raised

### Build System
- **Randomized fixed-literal residue** across 8 token families
- **HWID locking** accepts static fingerprint format alongside dynamic
- **Opt-in renaming** of proven-internal public methods and attributes
- **Diagnostic build mode** reports per-stage timings, transform counts, key-material freshness check; artifact is visibly marked non-release
- **Private build manifest** optional, records build provenance
- **Cross-compilation** native protection layer for Windows buildable from Linux

## Performance (v10.0)

| Family | Overhead vs CPython 3.12 |
|---|---|
| Import | 0.37x |
| Container | 0.86x - 0.94x |
| Scalar | 0.81x - 0.82x |
| Pure-compute (lowered) | 1.2x - 1.6x |

Pure-compute lowered calls dropped from the previous `2x–6x` range after dead tamper-guard wrapper removal.

**General**
- Compiled kernel exports randomized per build
- Runtime names resolved per-process, not emitted as literals
- Kernel bootstrap uses split-secret + fragmented decoder (no single embedded key)
- Payload split into sealed sections, streamed one at a time
- Key and white-box pages pinned; sections re-protected after load
- Single failure path: tamper trigger → wipe key material → exit
- Self-corruption replaces self-delete; detached child process handles overwrite; opt-out available
- Byte-tamper sweep: 12 mutations, two randomized builds, all fail-closed
- Native watchdog thread with heartbeat + timeout kill
- Crash-dialog suppression
- Dead tamper-guard wrapper stripped from pure-compute lowered calls → overhead down to `1.2x–1.6x`
- Dedicated regression test for every bug fixed in this release

**Features**
- 14 lowered builtin families including I/O
- I/O, scalar, string, container, service, protocol, function, import, fusion lowering on by default
- Arithmetic, truth, slice, format, augmented-assignment as opt-in per-family
- Per-family lowering opt-outs (including I/O)
- 141 native operations exposed by kernel
- Nested lowered call fusion + f-string fast path
- Guard pages for sealed sections
- Native anti-debug tier (debug register, debug port, toolchain checks)
- Audit-hook timing + hook-addition detection + site-packages hook detection
- Timing side-channel detector (clock-interrupt bias)
- Randomized fixed-literal residue across 8 token families
- HWID locking: static fingerprint format support added
- Opt-in renaming of proven-internal public methods/attributes
- Diagnostic build mode (per-stage timings, transform counts, key freshness; non-release mark)
- Optional private build manifest
- Cross-compile native layer for Windows from Linux
