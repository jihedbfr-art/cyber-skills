---
format: "v2"
name: "delegation-abuse"
title: "Delegation Abuse"
title_fr: "Abus de délégation Kerberos"
description: "Use when testing Kerberos delegation configurations in AD — unconstrained, constrained, and resource-based — for the paths they open to impersonation and domain compromise, plus the fixes."
description_fr: "À utiliser pour tester les configurations de délégation Kerberos dans AD — sans contrainte, contrainte et basée sur les ressources — afin d'identifier les chemins d'usurpation et de compromission du domaine qu'elles ouvrent, et les corrections associées."
domain: "12-active-directory-and-windows-security"
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

Kerberos delegation lets a service act on behalf of a user toward other services — a legitimate need (a web app reaching a database as you), but a dangerous one when misconfigured. Each delegation type has abuse paths that let an attacker impersonate users, including domain admins, and pivot to full compromise. This skill covers recognising and abusing the three delegation types, and configuring delegation so it can't be turned against you.

### When to use it

Mid-to-late AD engagements, after enumeration (BloodHound flags delegation) points you at delegated accounts. Delegation abuse is a common route from a service-account foothold to domain compromise, so it's high-value to understand on both sides.

### The three types (and their abuse)

- **Unconstrained delegation** — the service can impersonate a user to *any* service. When a user (or a coerced machine account, e.g. a DC) authenticates to an unconstrained-delegation host, their TGT is cached there — steal it and impersonate them anywhere. The most dangerous type; a DC's TGT means domain compromise.
- **Constrained delegation** — the service can impersonate users only to *specific* services (a defined list). Abuse: if you control the account, you can request tickets to those target services as any user (including via the S4U extensions), and protocol transition can let you do so without the user ever authenticating.
- **Resource-based constrained delegation (RBCD)** — the *target* resource decides who may delegate to it. Abuse: if you can write to a computer object's `msDS-AllowedToActOnBehalfOfOtherIdentity` (a common ACL/relay outcome), you configure an account you control to impersonate any user to that machine.

### Procedure

1. **Enumerate delegation config.** BloodHound highlights unconstrained/constrained/RBCD; or query LDAP for the relevant `userAccountControl` flags and `msDS-AllowedToDelegateTo` / `msDS-AllowedToActOnBehalfOfOtherIdentity` attributes.
2. **Unconstrained:** if you control an unconstrained-delegation host, capture TGTs of users who authenticate to it. Combine with coercion to force a high-value account (a DC machine account) to authenticate, then extract and reuse its ticket:
   ```
   Rubeus.exe monitor /interval:5      # watch for cached TGTs
   ```
3. **Constrained:** with control of a constrained-delegation account, use S4U to request a service ticket impersonating a chosen user to an allowed target:
   ```
   Rubeus.exe s4u /user:svc$ /rc4:<hash> /impersonateuser:admin /msdsspn:cifs/target
   ```
4. **RBCD:** if you can write `msDS-AllowedToActOnBehalfOfOtherIdentity` on a target computer, set it to an account you control, then S4U to impersonate any user to that machine:
   ```
   # set RBCD, then:
   getST.py -spn cifs/target -impersonate admin -dc-ip <dc> 'dom/attacker$:pass'
   ```
5. **Establish impact** — which users you can impersonate to which services, and whether that reaches domain-critical systems. Report the exact delegation object and the path.

### Cheatsheet

```
type            abuse                                    severity
--------------  ---------------------------------------  ------------------------
unconstrained   user auths -> their TGT cached -> steal  CRITICAL (coerce a DC ->
                & impersonate ANYWHERE                     domain compromise)
constrained     control acct -> S4U -> impersonate any   HIGH (to the allowed
                user to the ALLOWED services               targets)
RBCD            write msDS-AllowedToActOnBehalfOf... on   HIGH (common via ACL/
                target -> impersonate any user to it        relay outcomes)

enumerate: BloodHound (delegation edges) / LDAP UAC flags +
  msDS-AllowedToDelegateTo / msDS-AllowedToActOnBehalfOfOtherIdentity
tools: Rubeus (monitor, s4u), Impacket getST.py, findDelegation.py
```

### Reading the output

- **An unconstrained-delegation host you control** = you can harvest the TGT of anyone (or any coerced machine) that authenticates to it; coercing a DC turns this into domain compromise. The highest-severity delegation finding.
- **A constrained-delegation account under your control** = impersonation to its allowed targets as any user — scope the impact by what those targets are (CIFS on a file server vs a DC).
- **Write access to a computer's RBCD attribute** = you can configure impersonation to that machine; a frequent endpoint of ACL abuse and NTLM relay. High.
- **Protocol transition enabled (`TRUSTED_TO_AUTH_FOR_DELEGATION`)** = the account can impersonate users who never even authenticated — a broader abuse. Note it.
- **Delegation scoped tightly to non-sensitive services with no write exposure** = the lower-risk state; still document why the delegation exists.

### The fix

- **Eliminate unconstrained delegation.** There's almost never a justified modern use; replace it with constrained/RBCD scoped tightly. Unconstrained delegation on anything is a finding.
- **Protect sensitive accounts from delegation.** Mark privileged accounts "sensitive and cannot be delegated" (or add them to the Protected Users group) so their tickets can't be delegated even if a delegation path exists — this breaks the DC/admin-impersonation chain.
- **Scope constrained delegation minimally** — only the specific services required, and avoid protocol transition unless genuinely needed.
- **Lock down RBCD write access.** The `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute and computer-object ownership must be tightly controlled; combine with the NTLM-relay fixes (signing/channel binding), since relay is a common way attackers gain the write.
- **Monitor** S4U ticket requests and delegation attribute changes as detection signals.

### Pitfalls

- **Leaving unconstrained delegation "because it works".** It's a domain-compromise primitive waiting for a coerced DC. Migrate off it.
- **Not protecting privileged accounts from delegation.** Even tightly-scoped delegation becomes catastrophic if a Domain Admin's ticket can be delegated. Mark them sensitive / Protected Users.
- **Ignoring write access to RBCD attributes.** An ACL misconfiguration or a relay that grants that write is a full impersonation path to the target machine.
- **Overlooking protocol transition.** It removes the "the user must authenticate first" barrier, widening abuse — enable it only when required.

### References

- MITRE ATT&CK — T1558 (Steal or Forge Kerberos Tickets), T1187
- Rubeus and Impacket S4U documentation
- Microsoft — Kerberos delegation, Protected Users group, "sensitive and cannot be delegated"
- SpecterOps research on constrained/resource-based delegation abuse

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.