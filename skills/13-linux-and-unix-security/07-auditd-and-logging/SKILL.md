---
format: "v2"
name: "auditd-and-logging"
title: "Auditd And Logging"
title_fr: "Journalisation avec auditd"
description: "Use when setting up Linux audit logging — configuring auditd to record the security-relevant events that let you detect and investigate compromise, without drowning in noise."
description_fr: "À utiliser pour mettre en place la journalisation d'audit Linux — configurer auditd afin d'enregistrer les événements pertinents pour la sécurité, permettant de détecter et d'investiguer une compromission sans se noyer dans le bruit."
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

When something happens on a Linux host — a privilege escalation, a tampered config, a suspicious binary run — the audit log is how you know. The Linux audit daemon (`auditd`) records kernel-level security events, but out of the box it captures little of value, and configured naively it drowns you in noise. This skill covers configuring auditd to record the events that matter for detection and forensics, so a compromise leaves a trail you can actually follow.

### When to use it

Hardening a host for detectability, preparing for incident response (you can only investigate what was logged), or building the Linux side of a detection pipeline (feeds SOC/SIEM and threat hunting). Set it up *before* an incident — you can't retroactively log what already happened.

### Procedure

1. **Decide what to record — driven by threats, not volume.** Log the security-relevant events: authentication and privilege changes, execution of sensitive commands, modification of critical files, and use of key syscalls. A curated rule set beats "log everything" (which floods and gets ignored).
2. **Use a proven baseline rule set** rather than writing from scratch. Well-known auditd rule sets (e.g. the Neo23x0 auditd rules, or CIS-derived rules) map to real attacker behaviour and ATT&CK. Start from one and tune:
   ```
   # rules live in /etc/audit/rules.d/*.rules ; loaded by augenrules
   ```
3. **Cover the high-value events:**
   - **Identity/privilege:** changes to `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`, use of `sudo`/`su`, and privilege-changing syscalls (`setuid`, `setgid`).
   - **Critical file integrity:** watches on sensitive configs and binaries (`-w /etc/... -p wa`).
   - **Execution:** execve logging to record what commands ran (high value, higher volume — tune).
   - **Persistence surfaces:** cron, systemd units, SSH keys, kernel module loads.
4. **Tune out benign noise** — like IDS tuning, a few rules generate most of the volume; suppress the known-benign so the signal survives, without going blind on the security-relevant.
5. **Protect and ship the logs.** Audit logs on a compromised host can be tampered with — forward them off-host to a central store/SIEM promptly, so an attacker who gains root can't erase the evidence of how they got there. Make the local audit log append-restricted where possible.
6. **Know how to read it.** `ausearch` and `aureport` query the logs by key, user, time, or event type — practise querying before you need it in an incident.

### Cheatsheet

```bash
-w /etc/shadow -p wa -k identity          # watch shadow for write/attr change
-w /etc/sudoers -p wa -k sudoers
-w /etc/ssh/sshd_config -p wa -k sshd
-a always,exit -F arch=b64 -S execve -k exec     # command execution (tune volume)
-a always,exit -F arch=b64 -S setuid,setgid -k priv_change

augenrules --load ; auditctl -l

ausearch -k identity                       # by rule key
ausearch -ua <uid> -i                      # by user, interpreted
aureport --summary ; aureport -au          # summaries / auth report

Neo23x0 auditd ruleset | CIS-derived rules  -> tune, don't write from scratch

```

### Reading the setup

- **auditd off or default rules** = little security-relevant logging; a compromise leaves almost no trail. The gap this skill closes — deploy a real rule set.
- **"Log everything" with no tuning** = flooded logs nobody reads and a performance hit; the security events drown. Curate and tune (like IDS).
- **No file-integrity watches on critical files** = tampering with `/etc/shadow`, `sudoers`, or SSH config goes unrecorded — exactly the changes an attacker makes.
- **Logs kept only locally** = a root-level attacker wipes them, erasing how they got in. Forward off-host promptly — this is what makes the audit trail trustworthy.
- **execve logged but untuned** = high volume; valuable for forensics but needs tuning to stay manageable.
- **A tuned, threat-driven rule set forwarded to a SIEM** = the good state; compromises are detectable and investigable.

### The fix / best practice

- **Deploy a proven, threat-mapped rule set** (Neo23x0/CIS-derived) and tune it, rather than default rules or writing from scratch.
- **Watch the high-value targets** — identity/privilege files, sudo/su, critical configs and binaries, execution, and persistence surfaces (cron, systemd, SSH keys, module loads).
- **Tune out benign noise** so the signal survives and the logs stay usable.
- **Forward logs off-host promptly** to a central store/SIEM so an attacker who gains root can't erase the evidence — the single most important step for trustworthy auditing.
- **Feed the events into detection** (SOC/SIEM, detection-engineering) and practise `ausearch`/`aureport` before you need them.
- Combine with file-integrity monitoring for the critical-file watches.

### Pitfalls

- **Setting it up after the incident.** You can't log what already happened; auditd must be running and configured *before* compromise to be useful.
- **"Log everything."** Floods storage, hurts performance, and buries the signal. Curate by threat.
- **Keeping logs only on the host.** A root-level attacker deletes them — forward off-host so the trail survives the compromise.
- **Not tuning.** Untuned rule sets (especially execve) generate overwhelming volume; tune the noisy rules while keeping the security-relevant ones.
- **Never practising queries.** In an incident is the wrong time to learn `ausearch`; know it in advance.

### References

- auditd, auditctl, ausearch, aureport manuals
- Neo23x0 auditd ruleset and CIS Linux audit rules
- MITRE ATT&CK (map audit rules to techniques) and the detection-engineering/SOC skills
- NIST SP 800-92 (log management)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.