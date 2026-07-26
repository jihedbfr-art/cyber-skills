# 13 — Linux & Unix Security

The servers running your workloads. This domain covers hardening a host, the privilege-escalation tricks that turn a shell into root, and the auditing that tells you someone tried.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [linux-privilege-escalation](01-linux-privilege-escalation/SKILL.md) | Enumerate the ways up from a low-priv shell | ✅ |
| 02 | suid-sgid-audit | Find and reason about dangerous set-uid binaries | TODO |
| 03 | sudo-hardening | Lock down sudoers without breaking ops | TODO |
| 04 | file-permissions-and-acls | Get the basics actually right | TODO |
| 05 | systemd-hardening | Sandbox services with unit directives | TODO |
| 06 | ssh-hardening | Keys, config, and cutting the exposure | TODO |
| 07 | auditd-and-logging | Record what matters, find it later | TODO |
| 08 | selinux-apparmor | Mandatory access control that stays enforcing | TODO |
| 09 | kernel-and-sysctl-hardening | Sane sysctl and kernel settings | TODO |
| 10 | cis-benchmark-automation | Apply and verify a baseline at scale | TODO |

TODO: domain scaffolded. Suggested first skill: `linux-privilege-escalation`.
