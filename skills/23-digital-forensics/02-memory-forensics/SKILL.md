---
name: memory-forensics
domain: 23-digital-forensics
description: Use when analysing a memory image to find what disk forensics misses — running processes, injected code, network connections, and secrets that only exist in RAM.
difficulty: advanced
tags: [forensics, memory, volatility, ram, incident-response]
tools: [volatility, memprocfs]
---

## Purpose

A lot of what matters in an investigation never touches disk. Running processes, injected code, decryption keys, network connections, and fileless malware live only in memory — and vanish at power-off. Memory forensics analyses a RAM capture to recover them. This skill covers working a memory image with Volatility to surface what disk analysis can't, which is often where the answer to "what was this malware doing" actually lives.

## When to use it

After a memory capture (taken during evidence preservation — memory is the most volatile artifact, captured first). Essential for fileless malware, process injection, and any case where you need the *live* state of the system, not just what was written to disk.

## Procedure

1. **Start from a proper memory image.** Analysis is only as good as the capture — a full physical memory image acquired before power-off (the evidence-preservation skill covers acquisition). Work on a copy.
2. **Identify the profile/symbols.** Volatility needs to know the OS/kernel to interpret the image. Volatility 3 largely automates symbol resolution; older workflows required picking a profile. Get this right or nothing parses correctly.
3. **Enumerate processes — the starting point.** List running processes and look for anomalies: unexpected processes, suspicious parent-child relationships (Word spawning PowerShell), processes with no disk-backed image, or hidden processes:
   ```
   vol -f mem.raw windows.pslist          # process list
   vol -f mem.raw windows.pstree          # parent-child tree (spot odd chains)
   vol -f mem.raw windows.psscan          # scan for hidden/terminated processes
   ```
4. **Hunt for injection and malicious code.** Memory is where process injection and fileless malware live — find injected code, hollowed processes, and suspicious memory regions:
   ```
   vol -f mem.raw windows.malfind         # injected/executable private memory regions
   ```
5. **Recover network state** — active and recent connections show C2 and lateral movement that may not be logged anywhere else:
   ```
   vol -f mem.raw windows.netscan
   ```
6. **Extract secrets and artifacts** — command lines (what was run), loaded DLLs, registry data cached in memory, credentials, and sometimes encryption keys. Command-line history alone often reveals the attacker's actions:
   ```
   vol -f mem.raw windows.cmdline         # process command lines (high value)
   ```
7. **Dump suspicious processes/regions** for further analysis (feed into the malware domain) — a process dumped from memory is often the unpacked payload.

## Cheatsheet

```
prereq: full physical memory image (captured BEFORE power-off) — work on a copy
Volatility 3 auto-resolves symbols; get the OS/kernel right or nothing parses

process enumeration (start here)
  windows.pslist    running processes
  windows.pstree    parent-child (Word->PowerShell = suspicious)
  windows.psscan    hidden/terminated (scan, not just linked list)

malicious code / injection (memory-only findings)
  windows.malfind   injected/executable private memory -> process injection, fileless

network / secrets
  windows.netscan   active+recent connections (C2, lateral movement)
  windows.cmdline   command lines (what the attacker ran — high value)
  windows.dlllist / handles / hashdump / lsadump   loaded modules, creds

dump for deeper analysis
  windows.dumpfiles / procdump -> feed the malware domain (often the unpacked payload)

(Linux/macOS: equivalent linux.* / mac.* plugins)
```

## Reading the image

- **A suspicious process tree** (Office spawning a shell, a process with no disk image, an unusual parent) = a strong lead; process relationships in memory reveal execution chains that disk forensics can't show. Start here.
- **`malfind` hits** (executable private memory, injected code) = process injection or fileless malware — findings that exist *only* in memory and vanish at power-off. This is memory forensics' unique value.
- **Network connections to unexpected destinations** = C2 or lateral movement, often un-logged elsewhere; the live network state is memory-only.
- **Command lines showing attacker actions** = frequently the fastest route to understanding what happened — the exact commands run, decoded PowerShell, tools launched.
- **Credentials/keys recovered from memory** = how the attacker moved or what they decrypted; sometimes the key to encrypted evidence.
- **A hidden process (in psscan but not pslist)** = active hiding; the discrepancy between listing methods is itself the finding.

## Pitfalls

- **No memory capture, or captured too late.** Memory is the most volatile evidence; if it wasn't captured before power-off/reboot, it's gone. This is why evidence preservation captures RAM first — you can't do memory forensics without the image.
- **Wrong OS profile/symbols.** Volatility misinterprets the image and produces garbage or nothing; confirm the OS/kernel resolves correctly.
- **Only doing disk forensics.** Fileless malware, injection, and live network/credentials never hit disk; skipping memory misses an entire class of evidence.
- **Analysing the original capture.** Work on a copy, preserving the master (chain-of-custody applies to memory images too).
- **Stopping at process list.** The high-value findings are in malfind, cmdline, and netscan — go past enumeration to injection, actions, and network.

## References

- The Volatility Framework documentation (Volatility 3)
- The Art of Memory Forensics (the reference text)
- The evidence-preservation, disk-imaging-and-hashing, and malware domains
- MITRE ATT&CK — T1055 (Process Injection), fileless techniques
