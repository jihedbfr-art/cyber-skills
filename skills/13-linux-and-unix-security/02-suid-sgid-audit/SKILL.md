---
format: "v2"
name: "suid-sgid-audit"
title: "Suid Sgid Audit"
title_fr: "Audit des binaires SUID/SGID"
description: "Use when auditing a Linux host for SUID/SGID binaries — files that run with their owner's privileges — to find the ones that hand a low-priv user a path to root."
description_fr: "À utiliser pour auditer un hôte Linux à la recherche de binaires SUID/SGID — des fichiers exécutés avec les privilèges de leur propriétaire — afin d'identifier ceux qui ouvrent à un utilisateur peu privilégié un chemin vers root."
domain: "13-linux-and-unix-security"
tags: [cybersecurity, engineering, best-practices]
maturity: "stable"
audience: ["backend-engineer", "security-engineer", "coding-agent"]
requires: ["bash", "git"]
updated: "2026-08-08"
---



## Prerequisites
- Target system, dependencies and environment configured.

## Usage
### Purpose

A SUID binary runs with the privileges of its *owner*, not the user who launched it — so a SUID-root program executes as root no matter who runs it. That's necessary for a few system tools (`passwd` needs to write `/etc/shadow`), but every unnecessary SUID binary is a potential privilege-escalation path, and some can be tricked into spawning a root shell. This skill covers auditing the SUID/SGID set and removing what shouldn't be there.

### When to use it

Hardening a host, or as part of privilege-escalation enumeration (the linux-privilege-escalation skill points here). A clean SUID inventory is one of the highest-value Linux hardening steps because it closes a whole class of local escalation.

### Procedure

1. **Enumerate all SUID and SGID binaries** on the system:
   ```
   find / -perm -4000 -type f 2>/dev/null    # SUID
   find / -perm -2000 -type f 2>/dev/null    # SGID
   find / -perm -6000 -type f 2>/dev/null    # both
   ```
2. **Compare against the expected baseline.** A standard system has a known, small set of legitimate SUID binaries (`passwd`, `sudo`, `su`, `ping`, `mount`, `umount`, and a handful more). Anything outside that set — custom binaries, interpreters, editors, archive tools — is a candidate for concern.
3. **Cross-reference each non-standard binary against GTFOBins** — the reference for which common binaries can be abused when SUID to break out to a shell or read/write arbitrary files. A SUID binary that appears on GTFOBins is a direct escalation path:
   ```
   # e.g. SUID find -> find . -exec /bin/sh -p \; -quit  (root shell)
   # SUID vim/nano/less/cp/tar/nmap(old) etc. -> various escalations
   ```
4. **Investigate custom SUID binaries** — an in-house SUID program is a common weak point; if it calls other programs by relative path, runs a shell, or handles input unsafely, it's exploitable. These deserve scrutiny beyond the GTFOBins list.
5. **Check SGID too** — SGID on a binary or directory grants group privileges; less common as a root path but still a finding when it grants a sensitive group.
6. **Document the delta** — the binaries present that aren't in the expected baseline — since that delta is exactly what to remediate.

### Cheatsheet

```bash
find / -perm -4000 -type f 2>/dev/null    # SUID
find / -perm -2000 -type f 2>/dev/null    # SGID
find / -perm -u=s -o -perm -g=s -type f 2>/dev/null

find / -perm -4000 -type f -exec ls -la {} \; 2>/dev/null

in the standard baseline? (passwd, sudo, su, ping, mount, umount, ...)  -> ok
on GTFOBins as SUID-abusable?  -> escalation path  (gtfobins.github.io)
custom/in-house binary?        -> scrutinise (relative paths, shells, input)

chmod u-s /path/to/binary
```

### Reading the audit

- **A non-standard binary that appears on GTFOBins as SUID-abusable** = a confirmed local privilege-escalation path; a low-priv user gets a root shell from it. The top finding — remove the SUID bit.
- **A custom/in-house SUID binary** = high-scrutiny; these are frequently exploitable through relative-path calls (`PATH` hijack), unsafe input handling, or spawning a shell. Review the binary itself.
- **Interpreters or shells with SUID** (python, perl, bash, `env`) = near-instant root; there's rarely a legitimate reason and it's a critical finding.
- **A binary in the expected baseline** (`passwd`, `sudo`, `mount`…) = legitimate; leave it, though keep those patched.
- **A clean set matching the baseline** = the good state; the SUID escalation surface is minimal.

### The fix

- **Remove the SUID/SGID bit from anything that doesn't strictly need it** (`chmod u-s` / `chmod g-s`). The smaller the SUID set, the smaller the escalation surface — this is the direct remediation.
- **Eliminate SUID on interpreters and shells** immediately; there's essentially no valid reason for a SUID python/perl/bash.
- **Fix or remove exploitable custom SUID binaries** — if an in-house tool needs elevated privilege, prefer a tightly-scoped mechanism (a specific sudo rule, a capability, or a well-audited helper) over a broad SUID-root binary. Use absolute paths inside it and validate input.
- **Baseline and monitor** — record the legitimate SUID set and alert on new SUID binaries appearing (a new SUID file is a common persistence/escalation sign, and a detection opportunity).
- **Keep the legitimate SUID tools patched**, since a vulnerability in one (like the SUID-root tools hit by past CVEs) is a root exploit.

### Pitfalls

- **Only listing, never comparing.** The value is the *delta* from the expected baseline; a raw list without knowing what's legitimate doesn't tell you what to fix.
- **Missing SGID and interpreters.** Teams check for SUID-root shells but overlook SGID and SUID interpreters, which are just as exploitable.
- **Removing a needed SUID bit.** Stripping SUID from `passwd`/`sudo`/`mount` breaks the system; know the baseline before you `chmod`.
- **Ignoring custom binaries.** The GTFOBins list catches common tools, but your in-house SUID program may be the real hole — it needs its own review.
- **No monitoring.** A newly-appearing SUID binary is a red flag; without alerting, an attacker's SUID backdoor is invisible.

### References

- GTFOBins (gtfobins.github.io) — SUID abuse reference
- MITRE ATT&CK — T1548.001 (Setuid and Setgid)
- CIS Linux Benchmarks (SUID/SGID auditing)
- The linux-privilege-escalation skill (this is one of its enumeration steps)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.