---
name: ssh-hardening
domain: 13-linux-and-unix-security
description: Use when securing SSH access to Linux hosts — key-based auth, disabling weak options, and cutting the exposure of the most-attacked service on the internet.
difficulty: beginner
tags: [linux, ssh, hardening, authentication, remote-access]
tools: [sshd, ssh-keygen]
---

## Purpose

SSH is how you administer Linux hosts, and any SSH port exposed to the internet is brute-forced continuously. A hardened SSH config — keys instead of passwords, root login off, weak algorithms disabled — turns the most-attacked service on your hosts into a non-event. This skill covers the SSH server settings that matter and the exposure reduction around them.

## When to use it

Hardening any Linux host that runs SSH (nearly all of them), especially internet-facing ones. It's a quick, high-impact win — SSH brute-forcing and credential attacks are constant, and good config defeats them.

## Procedure

1. **Switch to key-based authentication and disable passwords — the core change.** Password auth invites brute-forcing; SSH keys don't. Generate a strong key, install the public key, confirm key login works, *then* disable password auth:
   ```
   ssh-keygen -t ed25519 -C "user@host"      # strong modern key
   ssh-copy-id user@host                       # install the public key
   # then in sshd_config:  PasswordAuthentication no
   ```
2. **Disable direct root login.** `PermitRootLogin no` forces admins to log in as a user and escalate (via sudo), removing the single most-targeted account from the login surface.
3. **Harden the sshd_config essentials:**
   - `PasswordAuthentication no` and `PubkeyAuthentication yes`
   - `PermitRootLogin no`
   - `PermitEmptyPasswords no`
   - restrict who can log in (`AllowUsers`/`AllowGroups`) to only the accounts that need SSH
   - disable unused features (`X11Forwarding no`, `AllowAgentForwarding no` unless needed)
4. **Restrict cryptographic algorithms** to strong ones — disable weak ciphers, MACs, and key-exchange algorithms (legacy CBC ciphers, SHA-1 MACs). Use a current recommended set so the connection can't be downgraded.
5. **Reduce exposure** — restrict source IPs at the firewall (SSH open only to a bastion/VPN/management range, not the whole internet), and consider fail2ban-style rate limiting to blunt brute-force noise. Changing the port is cosmetic (reduces log noise, not real risk) — don't rely on it.
6. **Validate and reload** — test the config, and keep an existing session open when reloading in case you locked yourself out:
   ```
   sshd -t                      # syntax-check before reload
   systemctl reload sshd
   ```

## Cheatsheet

```
core sshd_config hardening
  PasswordAuthentication no        # keys only (the big one)
  PubkeyAuthentication yes
  PermitRootLogin no               # no direct root
  PermitEmptyPasswords no
  AllowGroups sshusers             # restrict who can SSH
  X11Forwarding no
  # strong algorithms only (disable CBC ciphers, SHA-1 MACs, weak KEX)

keys
  ssh-keygen -t ed25519            # modern strong key
  ssh-copy-id user@host            # install pubkey; test BEFORE disabling passwords

exposure reduction
  firewall: SSH only from bastion/VPN/mgmt range, not 0.0.0.0/0
  rate-limit brute force (fail2ban) ; changing port = cosmetic only

safety: sshd -t (validate) ; keep a session open when reloading (avoid lockout)
```

## Reading the config

- **`PasswordAuthentication yes` on an internet-facing host** = open to continuous brute-forcing and credential stuffing; the top finding. Move to keys and disable passwords.
- **`PermitRootLogin yes`** = the most-targeted account is directly loginable; disable it and escalate via sudo instead.
- **Weak ciphers/MACs enabled** (CBC, SHA-1, legacy KEX) = downgrade-attackable; restrict to strong algorithms.
- **SSH open to the whole internet** = maximal exposure even with keys; restrict source IPs to a bastion/VPN/management range.
- **No login restriction** (`AllowUsers`/`AllowGroups` unset) = every account with a shell can SSH; scope it to those that need it.
- **Keys-only, no root login, strong algorithms, source-restricted, login-scoped** = the hardened state; SSH brute-forcing becomes a non-issue.

## The fix / best practice

- **Key-based auth, passwords disabled** — the single highest-impact SSH change. Ed25519 keys, password auth off after confirming key login works.
- **No direct root login** (`PermitRootLogin no`); admins log in as users and escalate.
- **Restrict who can log in** with `AllowUsers`/`AllowGroups`, and disable unused features (X11/agent forwarding).
- **Strong algorithms only** — a current recommended cipher/MAC/KEX set, weak legacy ones disabled.
- **Cut network exposure** — SSH reachable only from a bastion/VPN/management range at the firewall, not the open internet; add rate limiting.
- **Validate before reload and keep a session open** to avoid locking yourself out; consider MFA for SSH on high-value hosts.

## Pitfalls

- **Disabling passwords before confirming key login.** You'll lock yourself out. Test the key first, keep a session open, `sshd -t` before reload.
- **Relying on port-changing as security.** Moving off 22 cuts log noise but not real risk — attackers scan all ports. Do the real hardening.
- **Leaving root login on.** It's the account attackers target first; disable it.
- **Keys only, but exposed to the whole internet with weak algorithms.** Keys defeat brute-forcing, but restrict source IPs and algorithms too — defence in depth.
- **Forgetting to restrict who can SSH.** Every shell account being SSH-capable widens the surface; scope it.

## References

- sshd_config(5) manual
- Mozilla OpenSSH security guidelines (recommended algorithms)
- CIS Linux Benchmarks (SSH server configuration)
- MITRE ATT&CK — T1021.004 (Remote Services: SSH)
