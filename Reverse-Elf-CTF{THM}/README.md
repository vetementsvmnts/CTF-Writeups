<div align="center">

```
██████╗ ███████╗██╗   ██╗███████╗██████╗ ███████╗██╗███╗   ██╗ ██████╗     ███████╗██╗     ███████╗
██╔══██╗██╔════╝██║   ██║██╔════╝██╔══██╗██╔════╝██║████╗  ██║██╔════╝     ██╔════╝██║     ██╔════╝
██████╔╝█████╗  ██║   ██║█████╗  ██████╔╝███████╗██║██╔██╗ ██║██║  ███╗    █████╗  ██║     █████╗
██╔══██╗██╔══╝  ╚██╗ ██╔╝██╔══╝  ██╔══██╗╚════██║██║██║╚██╗██║██║   ██║    ██╔══╝  ██║     ██╔══╝
██║  ██║███████╗ ╚████╔╝ ███████╗██║  ██║███████║██║██║ ╚████║╚██████╔╝    ███████╗███████╗██║
╚═╝  ╚═╝╚══════╝  ╚═══╝  ╚══════╝╚═╝  ╚═╝╚══════╝╚═╝╚═╝  ╚═══╝ ╚═════╝     ╚══════╝╚══════╝╚═╝
```

### `Reversing ELF` — TryHackMe Room Writeup

*Eight ELF binaries, eight flags — static analysis, dynamic debugging, and a couple of tricks the compiler didn't want you to see.*

[![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/room/reverselfiles)
[![Difficulty](https://img.shields.io/badge/Difficulty-Easy-brightgreen?style=for-the-badge)](#)
[![Category](https://img.shields.io/badge/Category-Reverse%20Engineering%20%7C%20ELF%20%7C%20Radare2-00FF41?style=for-the-badge)](#)
[![Tools](https://img.shields.io/badge/Tools-strings%20%7C%20CyberChef%20%7C%20r2%20%7C%20ltrace-ff2020?style=for-the-badge)](#)

<img src="https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Reverse-Elf-CTF%7BTHM%7D/Assets/01-Reverse-Elf-Room-Completion.png" alt="Reversing ELF Room Banner" width="800"/>

</div>

---

## `0x00` Overview

| Field | Detail |
|---|---|
| **Room** | Reversing ELF |
| **Platform** | TryHackMe |
| **Scenario** | Eight standalone ELF binaries (`crackme1`–`crackme8`), each gating a flag behind a different reverse-engineering hurdle |
| **Attack Surface** | Local ELF binaries — no network component |
| **Core Skillset** | Static analysis (`file`, `strings`, `rabin2`), encoding recovery (CyberChef/`base64`), dynamic analysis (`radare2`, `ltrace`, `gdb`), and basic anti-analysis logic (hidden `strcmp`, integer-overflow guard) |
| **Objective** | Extract execute permissions on each binary → identify the gating mechanism per crackme → recover or bypass the password/comparison logic → capture all eight flags |
| **Author** | Kitsana Thuekoh ([@vetementsvmnts](https://github.com/vetementsvmnts)) |

---

## `0x01` Methodology / Attack Chain

```mermaid
flowchart LR
    A[Download & chmod +x] --> B[crackme1: Direct Execution]
    B --> C[crackme2: strings -> Plaintext Password]
    C --> D[crackme3: strings -> Base64 Decode]
    D --> E[crackme4: r2 Breakpoint -> strcmp Bypass]
    E --> F[crackme5: Integer Overflow -> atoi Guard]
    F --> G[crackme6: Helper Function Analysis]
    G --> H[crackme7: r2 pdf @main Static Trace]
    H --> I[crackme8: r2 pdf @main Static Trace]
    I --> J[All 8 Flags Captured]

    style A fill:#0d1117,stroke:#00FF41,color:#00FF41
    style B fill:#0d1117,stroke:#00FF41,color:#00FF41
    style C fill:#0d1117,stroke:#00FF41,color:#00FF41
    style D fill:#0d1117,stroke:#00FF41,color:#00FF41
    style E fill:#0d1117,stroke:#ff2020,color:#ff2020
    style F fill:#0d1117,stroke:#ff2020,color:#ff2020
    style G fill:#0d1117,stroke:#00FF41,color:#00FF41
    style H fill:#0d1117,stroke:#00FF41,color:#00FF41
    style I fill:#0d1117,stroke:#00FF41,color:#00FF41
    style J fill:#0d1117,stroke:#00FF41,color:#00FF41
```

---

## `0x02` Setup — Permissions & Baseline Triage

Each binary was downloaded and made executable before any analysis began. A quick `file` pass against each confirmed architecture and linking, which shaped the tool choice (static `strings`/`rabin2` first, `radare2`/`ltrace` only when a comparison was hidden).

```bash
chmod +x crackme1 crackme2 crackme3 crackme4 crackme5 crackme6 crackme7 crackme8
file crackme1
```

---

## `0x03` crackme1 — Direct Execution

No gating logic at all — the binary prints the flag as soon as it's run. A useful sanity check that the environment and permissions were set up correctly before moving into the harder binaries.

```bash
./crackme1
```

<p align="center">
  <img src="https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Reverse-Elf-CTF%7BTHM%7D/Assets/02-crackme1.png" alt="crackme1 Execution" width="800"/>
</p>

---

## `0x04` crackme2 — Plaintext Password via `strings`

Running the binary without arguments revealed it expected a password. Rather than guessing, `strings` was run directly against the binary, which surfaced the password sitting in plaintext in the `.rodata` section.

```bash
strings crackme2 | grep -i pass
./crackme2 <recovered_password>
```

<p align="center">
  <img src="https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Reverse-Elf-CTF%7BTHM%7D/Assets/03-crackme2.png" alt="crackme2 Password Recovery" width="800"/>
</p>

---

## `0x05` crackme3 — Base64-Encoded Flag (CyberChef)

`strings` again surfaced useful output, but this time the recovered value wasn't the flag itself — it was Base64-encoded. **CyberChef**'s `From Base64` recipe was used to reverse the encoding and recover the actual flag text.

```bash
strings crackme3 | grep -iE '[A-Za-z0-9+/]{20,}={0,2}'
```

> **Takeaway:** Base64 blobs sitting in a binary's strings output are a strong signal to reach for CyberChef before assuming a value is the final answer.

<p align="center">
  <img src="https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Reverse-Elf-CTF%7BTHM%7D/Assets/04-crackme3.png" alt="crackme3 Strings Output" width="800"/>
  <br/>
  <img src="https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Reverse-Elf-CTF%7BTHM%7D/Assets/04-crackme3flag.png" alt="crackme3 Decoded Flag" width="800"/>
</p>

---

## `0x06` crackme4 — Hidden Comparison via `radare2`

This binary hinted that the password string was hidden and compared using `strcmp`, so a static `strings` pass came up empty. **radare2** was used to disassemble the binary, locate the `strcmp@plt` call, and set a breakpoint immediately before the comparison to inspect the register holding the expected value at runtime.

```bash
r2 -d crackme4
afl
db <addr_of_strcmp_call>
dc
px @ rdi   # inspect the expected password in memory
```

<p align="center">
  <img src="https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Reverse-Elf-CTF%7BTHM%7D/Assets/05-crackme4.png" alt="crackme4 radare2 Breakpoint Analysis" width="800"/>
</p>

---

## `0x07` crackme5 — Integer Overflow on the Password Check

The binary compares the numeric value of user input (converted via `atoi`) against `0xcafef00d`. That value is larger than a signed 32-bit `int` can hold, so it wraps to a negative number and can never be matched by direct input. The fix was to break at the comparison instruction in **radare2** and overwrite the register value directly with `0xcafef00d` before continuing execution.

```bash
r2 -d crackme5
db <addr_of_cmp>
dc
dr eax = 0xcafef00d
dc
```

<p align="center">
  <img src="https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Reverse-Elf-CTF%7BTHM%7D/Assets/06-crackme5.png" alt="crackme5 Integer Overflow Bypass" width="800"/>
</p>

---

## `0x08` crackme6 — Helper Function Analysis

`strings` and a quick run gave no easy win here — a hint pointed to reading the source logic instead. `radare2` was used to list functions (`afl`) and disassemble a helper routine handling the real validation logic, which was walked through instruction-by-instruction to reconstruct the expected input.

```bash
r2 -A crackme6
afl
pdf @ sym.my_secure_test
```

<p align="center">
  <img src="https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Reverse-Elf-CTF%7BTHM%7D/Assets/07-crackme6.png" alt="crackme6 Helper Function Disassembly" width="800"/>
  <br/>
  <img src="https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Reverse-Elf-CTF%7BTHM%7D/Assets/08-crackme6.png" alt="crackme6 Flag Capture" width="800"/>
</p>

---

## `0x09` crackme7 — Static Trace of `main`

With `radare2`'s `pdf @ main` the full control flow of the entry function was visible in one pass, making the comparison logic and expected value straightforward to read directly out of the disassembly without needing a live breakpoint.

```bash
r2 -A crackme7
pdf @ main
```

<p align="center">
  <img src="https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Reverse-Elf-CTF%7BTHM%7D/Assets/09-crackme7.png" alt="crackme7 Static Disassembly" width="800"/>
</p>

---

## `0x0A` crackme8 — Static Trace & Final Flag

The final binary followed the same `pdf @ main` static-trace approach as crackme7, closing out the room with the eighth and final flag.

```bash
r2 -A crackme8
pdf @ main
```

<p align="center">
  <img src="https://raw.githubusercontent.com/vetementsvmnts/CTF-Writeups/main/Reverse-Elf-CTF%7BTHM%7D/Assets/10-crackme8.png" alt="crackme8 Static Disassembly and Final Flag" width="800"/>
</p>

---

## `0x0B` MITRE ATT&CK Mapping

*Applied to the analyst's tradecraft rather than an adversary — mapped for portfolio consistency with the rest of the writeup series.*

| Tactic | Technique | ID |
|---|---|---|
| Discovery | Software Discovery (`file`, `strings`, `rabin2` triage) | T1518 |
| Defense Evasion (analyzed) | Obfuscated/Compressed Files or Information (Base64) | T1027 |
| Defense Evasion / Analysis | Deobfuscate/Decode Files or Information (CyberChef) | T1140 |
| Execution (analyst) | Command and Scripting Interpreter — Unix Shell | T1059.004 |
| Reconnaissance (analyst) | Debugger/Disassembler Analysis (radare2, ltrace) | T1592.002 |

---

## `0x0C` Lessons Learned

- **`strings` first, always.** Half of these binaries gave up their secret with zero disassembly required — cheap wins should be exhausted before reaching for a debugger.
- **Encoded ≠ hidden.** A Base64 blob sitting in `.rodata` is not protection; it's a five-second CyberChef job.
- **`strcmp` in the disassembly is a breakpoint invitation.** Once a comparison function shows up in a static trace, a runtime breakpoint just before it hands over both sides of the check for free.
- **Integer overflow is a validation bug, not just a memory-safety one.** A password check built on `atoi()` against a value outside signed `int` range can never be satisfied by direct input — the fix lives in the register, not the argument.
- **`pdf @ main` in radare2 is underrated.** For small binaries, a single static disassembly of the entry function is often faster than setting up a full dynamic debugging session.

---

<div align="center">

`root@target:~#` **Room completed — 8/8 flags captured**

*Writeup by [Kitsana Thuekoh](https://github.com/vetementsvmnts) · Offensive Security Portfolio*

</div>
