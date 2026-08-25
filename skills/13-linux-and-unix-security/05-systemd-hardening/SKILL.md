---
format: "v2"
name: "systemd-hardening"
title: "Systemd Hardening"
title_fr: "Durcissement systemd"
description: "Use when hardening Linux services with systemd unit directives — sandboxing a daemon so a compromise of it can't reach the rest of the system."
description_fr: "À utiliser pour durcir des services Linux via les directives d'unité systemd — confiner un démon en sandbox pour qu'une compromission ne puisse pas atteindre le reste du système."
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

Every service is a potential foothold, and by default a compromised daemon runs with broad access to the host. systemd can sandbox services declaratively — restricting their filesystem, capabilities, system calls, and network — so that owning the service doesn't mean owning the box. This skill covers using systemd's security directives to shrink each service's blast radius, a modern and underused layer of Linux hardening.

### When to use it

Hardening any systemd-managed service, especially internet-facing or high-risk ones (web servers, custom daemons). It complements the file-permissions and least-privilege work by confining the process itself, not just what it owns.

### Procedure

1. **Measure the current exposure.** systemd can score a unit's sandboxing, giving you a baseline and a checklist of what's not yet restricted:
   ```
   systemd-analyze security <service>            # 0.0 (safe) .. 10.0 (exposed)
   systemd-analyze security                       # all services, ranked
   ```
   A high score means the service runs largely unconfined; the report lists each unset protection.
2. **Apply the high-value directives** in a drop-in override (`systemctl edit <service>`), starting with the ones that most shrink the blast radius:
   - **Filesystem:** `ProtectSystem=strict` (read-only OS), `ProtectHome=true`, `ReadWritePaths=` only what it needs, `PrivateTmp=true` (isolated /tmp).
   - **Privilege:** run as a dedicated non-root `User=`, `NoNewPrivileges=true` (blocks SUID/sudo escalation from within), `CapabilityBoundingSet=` limited to needed capabilities (or empty).
   - **Kernel/system:** `ProtectKernelTunables=true`, `ProtectKernelModules=true`, `ProtectControlGroups=true`, `RestrictSUIDSGID=true`.
   - **Network:** `RestrictAddressFamilies=` to only what's used, `PrivateNetwork=true` if it needs no network.
   - **Syscalls:** `SystemCallFilter=@system-service` (allowlist a sane set), blocking dangerous syscall groups.
3. **Apply incrementally and test.** Sandboxing can break a service that legitimately needs an access you removed — add directives in stages, restart, and confirm the service still works. Logs (`journalctl`) show what a too-tight restriction blocked.
4. **Re-measure.** Re-run `systemd-analyze security <service>` and confirm the score dropped and the intended protections are now set.
5. **Prioritise by risk** — put the most effort into the most exposed and most-targeted services; a fully sandboxed internet-facing daemon is worth more than tightening an already-isolated one.

### Cheatsheet

```bash
systemd-analyze security                 # rank all services
systemd-analyze security nginx.service   # detailed checklist for one

systemctl edit myservice.service

User=myservice            NoNewPrivileges=true
ProtectSystem=strict      ProtectHome=true       PrivateTmp=true
ReadWritePaths=/var/lib/myservice
CapabilityBoundingSet=    RestrictSUIDSGID=true
ProtectKernelTunables=true  ProtectKernelModules=true  ProtectControlGroups=true
RestrictAddressFamilies=AF_INET AF_INET6      # or PrivateNetwork=true if none
SystemCallFilter=@system-service

systemctl daemon-reload && systemctl restart myservice
journalctl -u myservice        # see what a too-tight restriction blocked
```

### Reading the assessment

- **A high `systemd-analyze security` score** (toward 10) on an exposed service = it runs largely unconfined; a compromise reaches the whole host. The biggest opportunity — sandbox it.
- **A service running as root that doesn't need to** = the first thing to fix; a dedicated `User=` plus `NoNewPrivileges=true` alone dramatically cuts the blast radius.
- **No filesystem protection** (`ProtectSystem`/`ProtectHome` unset) = a compromised service can read/write across the system; `strict` + scoped `ReadWritePaths` confines it.
- **Broad capabilities / no syscall filter** = the process can do far more than its job needs; tighten `CapabilityBoundingSet` and `SystemCallFilter`.
- **A service broken after hardening** = a directive removed an access it genuinely needs; the journal shows which — loosen that one specifically rather than abandoning the sandbox.
- **A low score with the key protections set** = the service is well-confined; a compromise is contained.

### The fix / best practice

- **Sandbox every service, prioritising exposed ones**, using the high-value directives above in drop-in overrides so vendor units stay intact.
- **Run services as dedicated non-root users** with `NoNewPrivileges=true` — the single most impactful pair.
- **Confine the filesystem** (`ProtectSystem=strict`, scoped `ReadWritePaths`, `PrivateTmp`) so a compromise can't roam.
- **Drop capabilities and filter syscalls** to the minimum the service needs.
- **Iterate with measurement** — `systemd-analyze security` before and after, and test each change so hardening doesn't break the service.
- Combine with the broader baseline (cis-benchmark skill) and keep the services patched.

### Pitfalls

- **Hardening blindly and breaking the service.** Over-tight directives block legitimate access; apply incrementally, test, and read `journalctl` to see what a restriction blocked before loosening exactly that.
- **Leaving services as root.** Much of the value is lost if the process still runs as root; a dedicated user plus `NoNewPrivileges` is the baseline.
- **Editing the vendor unit directly.** Use `systemctl edit` drop-ins so package updates don't overwrite your hardening.
- **Not measuring.** Without `systemd-analyze security` you're guessing; use it to prioritise and to verify the change landed.
- **Sandboxing already-isolated services while exposed ones stay open.** Prioritise by real risk.

### References

- systemd.exec(5) and systemd.resource-control(5) manuals (security directives)
- `systemd-analyze security` documentation
- Arch/Red Hat systemd hardening guides
- The file-permissions and cis-benchmark-automation skills

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.