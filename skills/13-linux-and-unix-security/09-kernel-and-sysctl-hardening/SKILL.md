---
name: kernel-and-sysctl-hardening
domain: 13-linux-and-unix-security
description: Use when hardening the Linux kernel via sysctl and boot settings — the network and memory-protection tunables that reduce exploitability and blunt common attacks.
difficulty: intermediate
tags: [linux, kernel, sysctl, hardening, exploit-mitigation]
tools: [sysctl]
---

## Purpose

Below the services and files sits the kernel, and a handful of kernel tunables (`sysctl`) meaningfully change a host's resilience — making exploitation harder, restricting information that helps attackers, and blunting network-level attacks. This skill covers the sysctl and kernel settings worth applying, grouped by what they protect, so you harden the base layer without cargo-culting a giant config you don't understand.

## When to use it

Hardening a host as part of a baseline, especially servers. These settings are low-effort (a config file) and complement the higher layers (services, MAC, permissions). Apply them from a benchmark and understand what each does rather than pasting an opaque list.

## Procedure

1. **Apply settings from a trusted baseline** (CIS, kernel-hardening projects) rather than inventing them, but understand each group so you can tell which apply to your host:
2. **Exploit-mitigation / information-restriction settings:**
   - `kernel.kptr_restrict=2` — hide kernel pointers (which help exploits bypass KASLR).
   - `kernel.dmesg_restrict=1` — restrict kernel log access (leaks info to non-root).
   - `kernel.yama.ptrace_scope=1` (or higher) — restrict `ptrace`, limiting one process from inspecting/injecting into another (blunts some credential-theft and injection).
   - `kernel.unprivileged_bpf_disabled=1` and restricting user namespaces where not needed — reduce kernel attack surface exposed to unprivileged users.
3. **Network-hardening settings:**
   - `net.ipv4.conf.all.rp_filter=1` — reverse-path filtering (anti-spoofing).
   - `net.ipv4.tcp_syncookies=1` — SYN flood mitigation.
   - `net.ipv4.conf.all.accept_redirects=0`, `send_redirects=0`, `accept_source_route=0` — ignore ICMP redirects and source routing (MITM/spoofing vectors).
   - disable IP forwarding (`net.ipv4.ip_forward=0`) unless the host is a router.
4. **Filesystem/process protections:**
   - `fs.protected_hardlinks=1`, `fs.protected_symlinks=1` — prevent a class of symlink/hardlink attacks in shared directories.
   - `fs.suid_dumpable=0` — don't allow SUID programs to dump core (which could leak secrets).
5. **Persist and apply.** Put settings in `/etc/sysctl.d/*.conf` so they survive reboot, and apply:
   ```
   # /etc/sysctl.d/60-hardening.conf  (then:)
   sysctl --system            # load all sysctl config
   sysctl -a | grep <key>     # verify a value took effect
   ```
6. **Test on your workload.** A few settings (ptrace scope, disabling namespaces/bpf) can break legitimate tools (debuggers, containers). Apply, then confirm your applications still work; loosen the specific setting if it genuinely conflicts.

## Cheatsheet

```
# apply via /etc/sysctl.d/*.conf  ->  sysctl --system

# exploit mitigation / info restriction
kernel.kptr_restrict=2
kernel.dmesg_restrict=1
kernel.yama.ptrace_scope=1
kernel.unprivileged_bpf_disabled=1

# network anti-spoofing / flood
net.ipv4.conf.all.rp_filter=1
net.ipv4.tcp_syncookies=1
net.ipv4.conf.all.accept_redirects=0
net.ipv4.conf.all.send_redirects=0
net.ipv4.conf.all.accept_source_route=0
net.ipv4.ip_forward=0            # unless the host is a router

# filesystem / process
fs.protected_hardlinks=1
fs.protected_symlinks=1
fs.suid_dumpable=0

verify: sysctl -a | grep KEY    | source baseline: CIS / kernel-hardening projects
```

## Reading the state

- **Default kernel settings on a server** = several cheap protections unused; applying a hardening baseline is low-effort risk reduction. Not a vulnerability per se, but a hardening gap.
- **`ip_forward=1` on a non-router host** = the host will route packets it shouldn't, a potential pivot/bypass. Disable unless it's genuinely a router/gateway.
- **`ptrace_scope=0` (default)** = any process can ptrace another of the same user, aiding credential theft and injection; raising it blunts that (but may affect debuggers).
- **Redirects/source-routing accepted** = MITM and spoofing vectors left open; disable them.
- **`kptr_restrict`/`dmesg_restrict` unset** = kernel info that helps exploits is exposed to unprivileged users; restrict it.
- **A benchmark-aligned sysctl config applied and verified** = the hardened base; understood, not cargo-culted.

## The fix / best practice

- **Apply a trusted baseline** (CIS Linux Benchmark sysctl settings, or a reputable kernel-hardening config), understanding each group rather than pasting blindly.
- **Prioritise the network anti-spoofing/flood settings and the info-restriction settings** — high value, low risk of breaking things.
- **Apply exploit-mitigation settings** (ptrace scope, kptr/dmesg restrict, bpf/namespace limits) with testing, since a few can affect debuggers and containers.
- **Persist in `/etc/sysctl.d/`** and verify with `sysctl -a` that values took effect after reboot.
- **Keep the kernel itself patched** — sysctl hardening reduces exploitability, but an unpatched kernel with a known local-exploit CVE is still a root risk; patching is the primary control.
- Automate via the cis-benchmark skill so it's applied consistently and doesn't drift.

## Pitfalls

- **Cargo-culting a giant sysctl file.** Pasting an opaque config you don't understand can break things and gives false confidence. Apply from a benchmark and know what each setting does.
- **Breaking legitimate tools.** ptrace-scope, bpf, and namespace restrictions can break debuggers and containers; test your workload and loosen the specific conflicting setting, not the whole config.
- **Leaving `ip_forward` on.** A non-router host routing packets is an unnecessary pivot path.
- **Treating sysctl as a substitute for patching.** These reduce exploitability but don't fix kernel vulnerabilities; keep the kernel updated.
- **Not persisting/verifying.** Runtime `sysctl -w` changes vanish on reboot; use `/etc/sysctl.d/` and confirm the values loaded.

## References

- CIS Linux Benchmarks (kernel/sysctl parameters)
- Kernel Self Protection Project (kernsec.org)
- sysctl(8) and the relevant kernel documentation
- The cis-benchmark-automation skill (to apply and verify at scale)
