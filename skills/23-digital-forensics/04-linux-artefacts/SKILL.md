---
name: linux-artefacts
domain: 23-digital-forensics
description: Use when investigating a Linux host — the logs, shell history, cron, and filesystem artefacts that reveal what happened, where evidence lives differently than on Windows.
difficulty: intermediate
tags: [forensics, linux, logs, artefacts, incident-response]
tools: [log2timeline, grep]
---

## Purpose

Linux investigations follow the same goals as Windows — what ran, when, who did it, how they persisted — but the evidence lives in different places, and there's no registry to lean on. Servers are disproportionately Linux, so knowing where Linux hides its forensic artefacts matters for the systems attackers most want. This skill covers the key Linux artefacts and where to look.

## When to use it

Investigating a compromised Linux host — a breached server, a container host, a suspect workstation. It pairs with the Linux security domain (which covers the same artefacts from the hardening/detection side, e.g. auditd) and the disk/memory acquisition skills.

## The artefacts by question

**"What happened / who logged in?" (Logs — mostly `/var/log`)**
- **auth.log / secure** — authentication, sudo, SSH logins (success and failure) — the who-and-when core.
- **syslog / messages** — general system events, service activity.
- **wtmp / btmp / lastlog** — login records (`last`, `lastb` read these); btmp is failed logins.
- **journald** (systemd) — the binary journal; query with `journalctl`.
- **audit.log** (auditd, if enabled) — detailed syscall/file/command auditing (see the auditd skill).

**"What commands were run?"**
- **Shell history** (`~/.bash_history`, `.zsh_history`, per user, including root) — commands typed. Often the fastest window into attacker actions — but easily cleared or disabled.

**"What's persisting / scheduled?"**
- **cron** (`/etc/crontab`, `/etc/cron.*`, user crontabs), **systemd units/timers**, **`~/.ssh/authorized_keys`** (added keys = backdoor access), **rc/profile scripts**, and **startup services**.

**"What files / when?" (Filesystem)**
- **timestamps** (mtime/atime/ctime), and the inode change time (ctime) which is harder to fake than mtime.
- **/tmp, /dev/shm, /var/tmp** — common staging/working directories for attackers.

## Procedure

1. **Frame the question**, then go to the artefact that answers it — as on Windows, a question-driven approach beats trawling `/var/log` blindly.
2. **Work the authentication and command trail first** — auth.log (who got in, sudo use), then shell history (what they did). This pair often reconstructs the intrusion quickly:
   ```
   # (on the image) review /var/log/auth.log ; last -f /var/log/wtmp ; lastb
   # shell history for all users incl. root (~/.bash_history, /root/.bash_history)
   ```
3. **Check persistence surfaces** — cron, systemd units/timers, `authorized_keys`, and startup scripts for attacker additions (an added SSH key or a rogue cron job is a common Linux backdoor).
4. **Examine attacker working directories** — `/tmp`, `/dev/shm`, `/var/tmp` for dropped tools, and look for recently-modified files and SUID binaries (ties into the Linux SUID skill).
5. **Build a timeline** from filesystem timestamps and logs (log2timeline/plaso; the timeline-analysis skill) — correlating log events with file activity reconstructs the sequence.
6. **Watch for anti-forensics** — cleared/truncated logs, a wiped or disabled shell history (`HISTFILE` unset, history size zeroed), and timestamp manipulation. Absence of expected logs is itself evidence.
7. **Handle as evidence** — work from the image, preserve, and document (chain-of-custody applies).

## Cheatsheet

```
question -> artefact (Linux hides evidence differently; no registry)

WHO LOGGED IN / AUTH
  /var/log/auth.log | secure   SSH/sudo/logins (success+fail)
  wtmp / btmp / lastlog         login records ->  last / lastb
  journald                       journalctl -u ... / --since ...
  audit.log                      auditd detail (if enabled)

WHAT RAN
  ~/.bash_history / .zsh_history (per user + ROOT)   commands typed (fast, but clearable)

PERSISTENCE
  /etc/crontab, /etc/cron.*, user crontabs
  systemd units + timers
  ~/.ssh/authorized_keys   (added key = backdoor)
  rc.local / profile scripts / startup services

FILES
  timestamps (ctime harder to fake than mtime)
  /tmp /dev/shm /var/tmp   (attacker staging) ; recently-modified ; SUID binaries

timeline: log2timeline/plaso  ->  correlate logs + filesystem
anti-forensics: cleared logs / wiped history / timestomping = itself a finding
```

## Reading the artefacts

- **The auth.log + shell-history pair** = often the fastest reconstruction — who got in (SSH/sudo) and what they typed. On Linux this is frequently the core of the investigation.
- **An added `authorized_keys` entry or a rogue cron/systemd job** = the persistence mechanism/backdoor; exactly what IR must remove. Very common on compromised Linux servers.
- **Tools or scripts in `/tmp`, `/dev/shm`, `/var/tmp`** = attacker staging; `/dev/shm` especially (memory-backed) is a favourite. Recently-modified files there are strong leads.
- **Cleared or truncated logs, or a disabled/empty shell history** = anti-forensics; the *absence* of expected evidence (an empty auth.log, `HISTFILE` unset) is a finding pointing at deliberate cleanup.
- **ctime/mtime inconsistencies** = possible timestomping; ctime is harder to forge, so mismatches are a tell.
- **Correlated log + filesystem timeline** = the reconstructed intrusion sequence; no single artefact gives it.

## Pitfalls

- **Applying Windows habits.** No registry, different log locations, journald is binary — Linux evidence lives elsewhere. Know the Linux-specific artefacts.
- **Trusting shell history as complete.** It's easily cleared, disabled, or per-shell; its absence isn't innocence, and attackers routinely wipe it. Corroborate with auth.log and auditd.
- **Missing `/dev/shm` and memory-backed staging.** Attackers use it precisely because it's overlooked and volatile.
- **Overlooking added SSH keys.** An `authorized_keys` backdoor is quiet and common; check it explicitly.
- **Ignoring the value of missing logs.** Cleared/truncated logs and disabled history are evidence of anti-forensics, not gaps to shrug at.
- **Analysing the live original.** Work from the image; live access changes atimes and can destroy evidence.

## References

- SANS Linux/Unix forensics resources
- log2timeline/plaso documentation
- The auditd-and-logging, disk-imaging-and-hashing, timeline-analysis, and anti-forensics-awareness skills
- MITRE ATT&CK (Linux persistence: T1053 cron, T1098.004 SSH keys)
