# 13 — Linux & Unix Security

The servers running your workloads. This domain covers hardening a host, the privilege-escalation tricks that turn a shell into root, and the auditing that tells you someone tried.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [linux-privilege-escalation](01-linux-privilege-escalation/SKILL.md) | Enumerate the ways up from a low-priv shell | ✅ |
| 02 | [suid-sgid-audit](02-suid-sgid-audit/SKILL.md) | Find and reason about dangerous set-uid binaries | ✅ |
| 03 | [sudo-hardening](03-sudo-hardening/SKILL.md) | Lock down sudoers without breaking ops | ✅ |
| 04 | [file-permissions-and-acls](04-file-permissions-and-acls/SKILL.md) | Get the basics actually right | ✅ |
| 05 | [systemd-hardening](05-systemd-hardening/SKILL.md) | Sandbox services with unit directives | ✅ |
| 06 | [ssh-hardening](06-ssh-hardening/SKILL.md) | Keys, config, and cutting the exposure | ✅ |
| 07 | [auditd-and-logging](07-auditd-and-logging/SKILL.md) | Record what matters, find it later | ✅ |
| 08 | [selinux-apparmor](08-selinux-apparmor/SKILL.md) | Mandatory access control that stays enforcing | ✅ |
| 09 | [kernel-and-sysctl-hardening](09-kernel-and-sysctl-hardening/SKILL.md) | Sane sysctl and kernel settings | ✅ |
| 10 | [cis-benchmark-automation](10-cis-benchmark-automation/SKILL.md) | Apply and verify a baseline at scale | ✅ |

This domain is complete (10/10). Start with `linux-privilege-escalation` (the attacker's view); `cis-benchmark-automation` ties the hardening skills into an operable baseline.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>