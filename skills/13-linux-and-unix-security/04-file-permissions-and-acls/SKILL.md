---
format: "v2"
name: "file-permissions-and-acls"
title: "File Permissions And Acls"
title_fr: "Permissions de fichiers et ACL"
description: "Use when reviewing Linux file and directory permissions — the world-writable files, exposed sensitive files, and misused ACLs that lead to tampering or privilege escalation."
description_fr: "À utiliser pour revoir les permissions de fichiers et de répertoires sous Linux — fichiers accessibles en écriture à tous, fichiers sensibles exposés et ACL mal utilisées, qui mènent à de la falsification ou à une élévation de privilèges."
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

Unix permissions are the foundation of Linux security, and the basics are where a surprising number of real problems live: a world-writable script that root runs, a readable private key, a sensitive config anyone can open. Get the fundamentals right and you close a whole layer of tampering and escalation. This skill covers auditing permissions (and POSIX ACLs) for the mistakes that matter.

### When to use it

Hardening a host, reviewing a system after a finding, or as part of privilege-escalation enumeration (writable files that root uses are a classic path). It's basic, which is exactly why it gets overlooked — and why it keeps causing incidents.

### Procedure

1. **Find world-writable files and directories — the highest-value check.** A world-writable file that a privileged process reads or executes is a direct escalation/tampering path:
   ```
   find / -type f -perm -0002 ! -path '/proc/*' 2>/dev/null     # world-writable files
   find / -type d -perm -0002 ! -path '/proc/*' 2>/dev/null     # world-writable dirs
   ```
2. **Focus on world-writable files in sensitive contexts** — anything in a system path, a script run by root/cron, a config a service reads. A world-writable cron script or a config a daemon trusts is root compromise waiting to happen.
3. **Find exposed sensitive files — readable secrets.** Private keys, credential files, and configs with passwords should be tightly restricted (`600`/`640`, correct owner). Check the ones that commonly leak:
   ```
   find / -name 'id_rsa' -o -name '*.pem' -o -name '.env' 2>/dev/null    # then check perms
   ls -la /etc/shadow                # should be 640 root:shadow (or 000/600), never world-readable
   ```
4. **Check world-writable directories for the sticky bit.** Shared-writable dirs like `/tmp` must have the sticky bit (`+t`, shows as `drwxrwxrwt`) so users can't delete/rename each other's files; a world-writable dir *without* it is a hijacking risk.
5. **Review ACLs where used.** POSIX ACLs (`getfacl`) grant permissions beyond the owner/group/other model — and can silently widen access. A file that looks `640` may grant read to others via an ACL:
   ```
   getfacl /path/to/file            # check for unexpected extra grants
   find / -type f -exec ls -la {} \; 2>/dev/null | grep '+'   # files with ACLs (the '+' flag)
   ```
6. **Check ownership**, not just mode — a sensitive file owned by a non-root user, or a service file owned by a user account, may be modifiable by the wrong party.

### Cheatsheet

```bash
find / -type f -perm -0002 ! -path '/proc/*' 2>/dev/null
find / -type d -perm -0002 ! -path '/proc/*' 2>/dev/null

ls -la /etc/shadow            # not world-readable
find / \( -name 'id_rsa' -o -name '*.pem' -o -name '.env' \) 2>/dev/null   # check perms

ls -ld /tmp /var/tmp

getfacl /path/file
ls -la ... | grep '+'          # the trailing + = an ACL is present

world-writable file that root/cron/a service uses  -> escalation/tampering
readable private key / credential file             -> secret exposure
world-writable dir without sticky bit              -> file hijacking
```

### Reading the audit

- **A world-writable file that a privileged process runs or reads** (cron script, service config, a binary root executes) = a direct root-compromise path; the highest-severity permission finding. Fix the mode immediately.
- **A readable private key or credential file** (`id_rsa`, `.pem`, `.env`, config with a password readable by others) = secret exposure; anyone on the host can take it. Restrict to owner-only.
- **A world-writable directory without the sticky bit** = users can delete/replace each other's files there; a hijacking and sometimes escalation vector.
- **An ACL granting access the visible mode hides** = the file is more exposed than `ls -l` suggests; ACLs are easy to overlook and can silently widen access.
- **A sensitive file owned by the wrong user** = modifiable by an unexpected party even if the mode looks restrictive.
- **Tight, correctly-owned permissions with sticky bits on shared dirs** = the good baseline.

### The fix

- **Remove world-writable permissions** from anything sensitive — especially files privileged processes use. `chmod o-w`, and set the minimum needed mode.
- **Lock down secrets** — private keys and credential files to `600` (owner-only), shared service configs to `640` with the right owner/group. `/etc/shadow` must not be world-readable.
- **Set the sticky bit** on shared-writable directories (`chmod +t /tmp`) so users can't tamper with each other's files.
- **Review and minimise ACLs** — remove ACL grants that widen access unnecessarily; check with `getfacl`, since they hide from the standard mode.
- **Fix ownership** so sensitive and service files are owned by the correct (usually root or a dedicated service) account.
- **Baseline and monitor** — a CIS-style baseline (see the cis-benchmark skill) enforces most of this, and file-integrity monitoring catches permission changes.

### Pitfalls

- **Overlooking the basics.** Permissions feel too fundamental to be the problem, so they get skipped — yet world-writable files and readable keys cause real incidents. Audit them.
- **Ignoring ACLs.** A file can look `640` and still grant read to others via an ACL; `ls -l`'s trailing `+` is the only hint. Check `getfacl`.
- **Missing the sticky bit on shared dirs.** World-writable `/tmp`-style directories without `+t` let users hijack each other's files.
- **Fixing the mode but not the ownership.** A restrictive mode on a file owned by the wrong user still lets that user modify it.
- **World-writable files that root uses.** The single most dangerous case — treat any writable file in a privileged process's path as a root exposure.

### References

- CIS Linux Benchmarks (filesystem permissions)
- MITRE ATT&CK — T1222 (File and Directory Permissions Modification)
- chmod(1), getfacl(1), setfacl(1) manuals
- The linux-privilege-escalation and cis-benchmark-automation skills

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.