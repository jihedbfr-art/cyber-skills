---
name: linux-privilege-escalation
domain: 13-linux-and-unix-security
description: Use when you have a low-privilege shell on a Linux host and need to enumerate the ways up to root — SUID, sudo, cron, capabilities — and how to close each.
difficulty: intermediate
tags: [linux, privilege-escalation, enumeration, hardening]
tools: [linpeas, gtfobins, sudo]
---

## Purpose

A low-priv shell is a starting point, not a dead end. Linux hosts accumulate misconfigurations — a permissive sudo rule, a writable cron script, an SUID binary that can spawn a shell — that turn a normal user into root. This skill covers enumerating those paths systematically and, for defenders, removing them.

## When to use it

After landing a shell on an authorised engagement, or auditing your own hosts for escalation exposure. The enumeration is the same either way; only the follow-up differs (exploit vs remediate).

## Procedure

1. Establish who and where you are, then enumerate broadly with an automated script before hand-hunting:
   ```
   id; uname -a; sudo -l
   ./linpeas.sh | tee linpeas.out
   ```
2. **sudo rules** are the fastest win. `sudo -l` shows what you can run as root. Any entry — especially `NOPASSWD` — check against GTFOBins for a shell escape:
   ```
   sudo -l
   # e.g. (root) NOPASSWD: /usr/bin/find  ->  sudo find . -exec /bin/sh \; -quit
   ```
3. **SUID/SGID binaries** run as their owner. Find non-standard ones and check GTFOBins for each:
   ```
   find / -perm -4000 -type f 2>/dev/null
   ```
4. **Capabilities** can grant root-like powers to specific binaries without SUID:
   ```
   getcap -r / 2>/dev/null
   # cap_setuid on python/perl -> instant root
   ```
5. **Writable cron jobs and scripts** running as root: if you can edit a script root executes on a schedule, you own root:
   ```
   cat /etc/crontab; ls -la /etc/cron.*; find / -writable -name '*.sh' 2>/dev/null
   ```
6. **Kernel and service versions** for known local exploits — but treat kernel exploits as a last resort (they crash boxes). Prefer a misconfig path if one exists.
7. Check the usual extras: readable `/etc/shadow`, credentials in config/history files, writable `/etc/passwd`, PATH hijacking on a root-run script that calls a binary by relative name.

## Cheatsheet

```bash
# fast triage
sudo -l                                  # sudo rights (check GTFOBins)
find / -perm -4000 -type f 2>/dev/null   # SUID binaries
getcap -r / 2>/dev/null                  # capabilities
find / -writable -type f 2>/dev/null | grep -vE '/proc|/sys'
cat /etc/crontab /etc/cron.d/* 2>/dev/null

# automated enumeration
./linpeas.sh          # broad, colour-coded by likelihood
./lse.sh -l1          # linux smart enumeration

# every SUID/sudo/cap escape lives here:
#   https://gtfobins.github.io
```

## Reading the output

- **A `sudo -l` entry that GTFOBins can turn into a shell** is the cleanest escalation — often a single command to root.
- **A non-standard SUID binary** (something that isn't `passwd`, `ping`, `mount`…) is a lead; cross-reference GTFOBins.
- **`cap_setuid`/`cap_setgid` on an interpreter** (python, perl) is effectively root in one line.
- **A root cron job calling a world-writable script** = reliable root on the next tick.
- linpeas highlighting in **red/yellow** flags the highest-probability paths — start there, but verify manually; it also flags noise.

## The fix

Each path has a specific remediation:

- **sudo:** grant the minimum, avoid `NOPASSWD`, and never grant sudo on binaries with a documented shell escape (editors, `find`, `vim`, interpreters). Audit `/etc/sudoers` against GTFOBins.
- **SUID/SGID:** remove the bit from anything that doesn't need it (`chmod u-s`). Keep the set small and known.
- **Capabilities:** drop capabilities that aren't required; never put `cap_setuid` on a scripting interpreter.
- **Cron:** root-run scripts must be root-owned and not group/world-writable; use absolute paths inside them.
- **Patch** the kernel and services so local exploits don't apply.
- **Baseline it:** the CIS Benchmark automation skill enforces most of this at scale so hosts don't drift back.

## Pitfalls

- **Reaching for kernel exploits first.** They're unreliable and can crash the host. Exhaust misconfig paths before touching a local kernel exploit — especially on production.
- **Trusting the scanner blindly.** linpeas flags possibilities, not confirmations; verify before reporting, and before firing anything destructive.
- **Missing capabilities.** Teams that lock down SUID often forget `getcap` — a `cap_setuid` binary is just as good as SUID root.
- **Fixing the binary, not the pattern.** One writable script fixed while the deploy keeps recreating it world-writable helps nothing. Fix the source.

## References

- GTFOBins (gtfobins.github.io)
- PEASS-ng / linpeas documentation
- CIS Benchmarks for Linux
- MITRE ATT&CK — T1548 (Abuse Elevation Control Mechanism)
