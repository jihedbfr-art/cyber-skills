---
name: selinux-apparmor
domain: 13-linux-and-unix-security
description: Use when applying mandatory access control on Linux — keeping SELinux or AppArmor enforcing (not disabled) so a compromised process is confined to what it's meant to do.
difficulty: advanced
tags: [linux, selinux, apparmor, mac, hardening, confinement]
tools: [selinux, apparmor, semanage]
---

## Purpose

Standard Unix permissions decide what a *user* can do; Mandatory Access Control (MAC) decides what a *process* can do, regardless of user — so even a root-running service is confined to a policy. SELinux and AppArmor are the two Linux MAC systems, and they turn a compromised web server from "attacker has the box" into "attacker is stuck in the web server's tiny sandbox". This skill covers keeping MAC enforcing and working with it rather than the common reflex of disabling it.

## When to use it

Hardening servers, especially internet-facing services, where confining a compromise matters most. The single most important message: **don't disable it** — the reflex to set SELinux permissive/disabled when something breaks throws away a major defence layer. This skill is about keeping it on and troubleshooting properly instead.

## The two systems

- **SELinux** (Red Hat/Fedora/CentOS default) — label-based: every process and file has a security context, and policy governs which contexts can interact. Powerful and fine-grained, with a reputation for complexity that leads people to disable it.
- **AppArmor** (Ubuntu/SUSE default) — path-based profiles per program: simpler to read and write, profiles confine a binary to declared files and capabilities.

Both achieve the same goal — confining processes to a policy. Which you use is usually decided by your distro.

## Procedure

1. **Confirm it's enforcing.** The first check, because the common "fix" is to turn it off:
   ```
   getenforce                       # SELinux: should say "Enforcing" (not Permissive/Disabled)
   aa-status                         # AppArmor: profiles loaded and in enforce mode?
   ```
   `Permissive` or `Disabled` SELinux, or unloaded/complain-mode AppArmor profiles, means the protection isn't actually protecting.
2. **Understand what's confined.** Check which services have enforcing policies/profiles. A service running unconfined gains nothing from MAC; the value is in the targeted, enforcing policies on your real services.
3. **Troubleshoot the right way when something breaks.** When a service fails under MAC, the reflex is to disable MAC — the correct move is to read the denial and fix the policy:
   ```
   # SELinux: denials are logged; interpret them
   ausearch -m avc -ts recent | audit2why       # why was it denied?
   # legitimate access -> set the right context / boolean, e.g.
   semanage fcontext / setsebool -P ...          # persistent fix
   # AppArmor: adjust the profile
   aa-logprof                                     # walk through denials, update profile
   ```
   Grant the specific access the service legitimately needs, not a blanket disable.
4. **Use booleans and contexts (SELinux) or profile edits (AppArmor)** for the common adjustments — most "SELinux is blocking my app" cases are a wrong file context or an unset boolean, fixed in one command, not a reason to disable it.
5. **Write/refine profiles for custom services** — AppArmor's `aa-genprof`/`aa-logprof` or SELinux policy tooling can build a confinement for an in-house daemon so it's sandboxed like the packaged ones.
6. **Keep it enforcing in production** — permissive mode is for troubleshooting/learning, not a resting state.

## Cheatsheet

```bash
# is it actually on? (the #1 check)
getenforce                 # SELinux -> want: Enforcing
sestatus                   # full SELinux status
aa-status                  # AppArmor -> profiles in enforce mode?

# troubleshoot the RIGHT way (don't disable!)
ausearch -m avc -ts recent | audit2why     # SELinux: why denied?
semanage fcontext -a -t <type> "/path(/.*)?"  && restorecon -Rv /path   # fix context
setsebool -P <boolean> on                   # flip a policy boolean (persistent)
aa-logprof                                   # AppArmor: update profile from denials

# custom service confinement
aa-genprof /usr/bin/mydaemon                 # AppArmor: build a profile
# (SELinux: audit2allow / policy modules for custom policy)

golden rule: a denial means "fix the policy", NOT "disable SELinux/AppArmor".
```

## Reading the state

- **SELinux `Permissive` or `Disabled` (or AppArmor profiles in complain/unloaded)** = the MAC layer is off; whatever confinement you thought you had isn't there. The most common and most important finding — turn it back on and fix the underlying policy issue properly.
- **Key services running unconfined** = MAC is enforcing but not applied where it matters; ensure your real (especially internet-facing) services have enforcing policies/profiles.
- **A history of "we disabled SELinux because it broke X"** = a fixable policy issue was resolved by removing the protection; the right fix is a context/boolean/profile change, and MAC should go back to enforcing.
- **AVC denials in the logs for legitimate access** = the policy needs a targeted grant (`audit2why`/`setsebool`/`semanage`), not a shutdown.
- **Enforcing MAC with targeted policies on all real services** = the strong state; a service compromise is confined to its policy.

## The fix / best practice

- **Keep it enforcing.** The core practice — never leave SELinux permissive/disabled or AppArmor in complain mode as a resting state in production.
- **Fix denials properly** — read the AVC/denial, grant the specific legitimate access via contexts/booleans (SELinux) or profile edits (AppArmor). Most issues are one command.
- **Confine your real services**, especially internet-facing ones; a compromised confined service is trapped in its sandbox.
- **Write profiles/policies for custom daemons** so in-house services get the same confinement as packaged ones.
- **Use permissive/complain mode only for troubleshooting and building policy**, then return to enforcing.
- Combine with systemd sandboxing and least privilege — MAC is another layer, not a replacement.

## Pitfalls

- **Disabling it at the first problem.** The near-universal mistake — a MAC denial gets "fixed" by turning MAC off, discarding a major defence. Fix the policy instead; the tools make it straightforward.
- **Leaving it permissive "temporarily".** Permissive logs but doesn't enforce; temporary becomes permanent and you're unprotected. Return to enforcing.
- **Assuming enforcing = everything confined.** Unconfined services gain nothing; confirm your real services have policies/profiles.
- **Blanket `audit2allow` without reading.** Auto-generating a policy that allows everything the service tried defeats the point; grant the legitimate access deliberately.
- **Ignoring it entirely.** An enforcing, well-configured MAC turns many compromises into contained non-events; skipping it forfeits that.

## References

- SELinux and AppArmor official documentation
- Red Hat SELinux troubleshooting guide (audit2why, semanage, booleans)
- MITRE ATT&CK (MAC as a mitigation for many execution/persistence techniques)
- The systemd-hardening skill (complementary process confinement)
