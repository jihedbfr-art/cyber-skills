---
format: "v2"
name: "sudo-hardening"
title: "Sudo Hardening"
title_fr: "Durcissement de sudo"
description: "Use when reviewing or writing sudoers configuration — granting elevated access without handing out shell escapes or effectively-root privileges by accident."
description_fr: "À utiliser pour rédiger ou revoir une configuration sudoers — accorder des accès élevés sans distribuer par mégarde des échappatoires shell ou des privilèges équivalents à root."
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

`sudo` is how Linux grants controlled elevation — but a loose sudoers rule quietly hands a user full root, because many innocuous-looking commands can spawn a shell once you can run them as root. `(root) NOPASSWD: /usr/bin/vim` isn't "let them edit files as root"; it's "give them a root shell". This skill covers writing and reviewing sudo rules so elevation stays scoped to what was intended.

### When to use it

Configuring sudo access, reviewing an existing sudoers for over-grants, or during privilege-escalation enumeration (`sudo -l` is the first thing to check on a foothold). It's a frequent, high-impact escalation path and a frequent hardening win.

### Procedure

1. **List what each user can run as root.** On a host you're assessing, `sudo -l` shows the current user's rights; for a full review, read `/etc/sudoers` and `/etc/sudoers.d/*`:
   ```
   sudo -l                      # what can I run elevated?
   sudo visudo -c               # syntax-check sudoers
   ```
2. **Flag the effectively-root grants first.** `ALL=(ALL) ALL` (or `NOPASSWD: ALL`) is full root — fine for a genuine admin, a finding for a service or regular account. The subtler danger is a *specific* command that's actually a shell escape.
3. **Check each allowed command against GTFOBins for sudo abuse.** A huge range of commands, when runnable as root via sudo, break out to a root shell — editors (`vim`, `nano`, `less`), interpreters (`python`, `perl`), and tools (`find`, `awk`, `tar`, `git`, `man`). If a sudo rule allows one of these, it's a root shell, not a scoped privilege:
   ```
   # e.g.  (root) NOPASSWD: /usr/bin/find
   #   ->  sudo find . -exec /bin/sh \; -quit     (root shell)
   ```
4. **Watch for wildcards and argument abuse.** A rule with `*` (e.g. `/bin/cp *`) often lets the user do far more than intended (overwrite `/etc/shadow`, copy arbitrary files). Wildcards in command arguments are a classic over-grant.
5. **Check `NOPASSWD` usage** — it removes the re-authentication barrier, so a hijacked session or a shell escape needs no password. Minimise it; require a password for elevation where feasible.
6. **Look for editable targets** — if a sudo rule runs a script the user can modify, or a binary in a user-writable path, that's a direct root path regardless of the command itself.

### Cheatsheet

```
review
  sudo -l                         # per-user elevated rights
  read /etc/sudoers + /etc/sudoers.d/*
  visudo -c                        # validate syntax (always edit with visudo)

red flags
  ALL=(ALL) ALL / NOPASSWD: ALL    -> full root (ok for admins, not services)
  a command on GTFOBins (sudo)     -> it's a ROOT SHELL, not a scoped privilege
       vim nano less find awk python perl tar git man sed ... (huge list)
  wildcards in args  /bin/cp *      -> often far more than intended
  NOPASSWD                          -> no re-auth barrier (shell escape = free root)
  sudo runs a user-editable script / binary in writable path -> direct root

principle: sudo grants the ABILITY of the command, including any shell it can spawn.
```

### Reading the review

- **A specific-command rule that's actually a GTFOBins shell escape** = the user has root, just indirectly; the most common sudo over-grant and easy to miss because the rule "looks scoped". Remove or replace it.
- **`NOPASSWD: ALL` on a non-admin or service account** = full passwordless root; critical. Service accounts especially should not have this.
- **Wildcards in command arguments** = the rule likely permits far more than intended (arbitrary file overwrite/copy). Tighten to exact arguments or a wrapper.
- **A sudo-runnable script the user can edit** = they edit it to do anything as root; the command doesn't even need a built-in escape.
- **Password-required, tightly-scoped rules to non-escaping commands** = the good state; elevation is real and bounded.

### The fix

- **Grant the minimum, and verify the command can't spawn a shell.** Before allowing a command via sudo, check GTFOBins — if it's abusable, don't grant it directly; wrap it in a purpose-built helper that does only the intended action.
- **Avoid `ALL` and wildcards** in command specs; specify exact commands and, where possible, exact arguments.
- **Require passwords for elevation** — minimise `NOPASSWD`, so a shell escape or hijacked session still needs authentication.
- **Never sudo-allow a user-editable script or a binary in a writable path** — lock down ownership and permissions of anything sudo runs.
- **Use `visudo`** to edit (it validates syntax and prevents lockout), and keep rules in `/etc/sudoers.d/` with clear ownership.
- **Audit `sudo -l` for key accounts** as part of hardening, and log sudo usage for detection.

### Pitfalls

- **Thinking a specific-command rule is safe.** `sudo vim` (or find, awk, less, python…) is a root shell. The command's *capabilities*, including shells it spawns, are what you grant — not just its nominal function.
- **Wildcards.** `/bin/cp *` and similar quietly permit far more than intended. Avoid `*` in command specs.
- **Overusing NOPASSWD.** It removes the last barrier before root; a shell escape then needs no password. Require re-auth where you can.
- **Sudo running editable content.** A script or binary the user can modify (or in a writable dir) turns any sudo rule into full root.
- **Editing sudoers without visudo.** A syntax error can lock everyone out of sudo; visudo validates first.

### References

- GTFOBins (gtfobins.github.io) — sudo abuse reference
- sudoers(5) manual
- MITRE ATT&CK — T1548.003 (Sudo and Sudo Caching)
- The linux-privilege-escalation skill (sudo -l is its first check)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.