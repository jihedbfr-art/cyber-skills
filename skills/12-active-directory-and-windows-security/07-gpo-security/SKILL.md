---
name: gpo-security
domain: 12-active-directory-and-windows-security
description: Use when assessing Group Policy as both an attack surface and a defensive tool — finding GPOs that can be abused to push code across the domain, and using them to enforce hardening.
difficulty: intermediate
tags: [active-directory, group-policy, gpo, windows, hardening]
tools: [bloodhound, powerview]
---

## Purpose

Group Policy is how AD pushes configuration to thousands of machines — which makes it a powerful hardening tool and a devastating attack vector at the same time. If an attacker can edit a GPO linked to sensitive systems, they can run code as SYSTEM across every machine it applies to. This skill covers both sides: finding abusable GPO permissions, and using Group Policy to enforce security at scale.

## When to use it

During AD assessment (who can modify which GPOs, and what they reach) and during hardening (deploying baseline settings domain-wide). Attack and defence share the same mechanism, which is why they're one skill.

## The attack: GPO edit rights as code execution

A GPO applies to the OUs, domains, or sites it's linked to. Anyone with edit rights on a GPO can add a scheduled task, a startup script, or a setting that runs as SYSTEM on every machine that processes it. So the security question is: **who can edit which GPOs, and what do those GPOs apply to?**

## Procedure

1. **Enumerate who can modify GPOs.** BloodHound flags `WriteGPO`/`GpLink` and dangerous GPO ACLs; PowerView can list GPO permissions. Look for non-admin principals with edit rights, especially on GPOs linked to sensitive OUs (domain controllers, servers):
   ```
   # BloodHound: "GPOs" and edit-rights edges; or PowerView:
   Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs | ? {$_.ActiveDirectoryRights -match 'Write'}
   ```
2. **Map GPO → target.** A GPO you can edit only matters as much as what it's linked to. An editable GPO applying to Domain Controllers is domain compromise; one applying to a test OU is minor. Trace the link.
3. **Understand the abuse (don't detonate on production).** With edit rights, an attacker adds an immediate scheduled task or startup script that runs as SYSTEM on affected machines. On an authorised test, demonstrate the *capability* (the edit right + the target) rather than pushing live code to the fleet — a malicious GPO hits every machine in scope.
4. **Check GPO-delivered credentials/settings** for exposure — historically, Group Policy Preferences stored passwords in a reversibly-encrypted `cpassword` (the MS14-025 issue); any lingering GPP passwords are a straight credential leak.
5. **Report the ACL and its reach** — the fix is the permission, not the payload.

## The defence: GPO as a hardening lever

6. **Use Group Policy to enforce the baseline** — Group Policy is how you deploy most of the AD hardening skill's settings at scale: disabling weak protocols, enforcing LSASS protection, setting audit policy, restricting local admin. One well-secured GPO hardens the whole domain.
7. **Protect the GPOs themselves.** Tightly control who can edit GPOs (especially those linked to Tier 0 / DCs), monitor GPO changes, and keep GPO edit rights inside the tiered-admin boundary.

## Cheatsheet

```
attack question: WHO can edit WHICH GPO, applying to WHAT?
  editable GPO linked to DCs/servers  -> SYSTEM code exec fleet-wide (critical)
  editable GPO linked to a test OU     -> minor

enumerate
  BloodHound: GPO edit-rights edges + GpLink to sensitive OUs
  PowerView:  Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs  (Write* rights)

abuse (capability, not live payload on prod)
  edit right -> immediate scheduled task / startup script -> runs as SYSTEM
  legacy: GPP cpassword in SYSVOL -> decryptable credentials (MS14-025)

defence
  deploy the hardening baseline via GPO (scale)
  lock down GPO edit rights (esp. DC-linked) + monitor GPO changes
```

## Reading the assessment

- **A non-admin with edit rights on a GPO linked to Domain Controllers or servers** = effectively SYSTEM code execution across those systems — a domain-compromise path. The highest-impact GPO finding.
- **`cpassword` in a Group Policy Preferences file in SYSVOL** = decryptable credentials readable by any domain user; a direct, well-known leak to remediate immediately.
- **Editable GPOs that only reach low-value OUs** = lower priority, but still an unintended permission worth cleaning up.
- **GPO edit rights held outside the admin tier** = a break in the tiered-admin model; that principal is effectively as privileged as what the GPO controls.
- **Well-scoped GPO permissions + a hardening baseline GPO in place** = the good state; Group Policy working for you, not against you.

## The fix

- **Restrict GPO edit rights** to the appropriate admin tier — nobody outside Tier 0 should edit GPOs linked to domain controllers. Audit GPO ACLs and remove unintended write access.
- **Remove any GPP `cpassword`** and rotate the exposed credentials; use LAPS for local admin passwords instead.
- **Monitor GPO changes** — creation, modification, and link changes are high-signal events; alert on them.
- **Use GPO deliberately for hardening** — deploy the baseline (protocol hardening, LSASS/Credential Guard, audit policy, restricted local admin) domain-wide, and keep those GPOs protected.
- Keep GPO management inside the tiered-admin model so editing a DC-linked policy requires Tier 0 access.

## Pitfalls

- **Assessing GPO edit rights without mapping the link.** The permission's severity is entirely about what the GPO applies to — an editable GPO on nothing is nothing; on the DC OU it's the domain.
- **Detonating a test payload on production GPOs.** A malicious GPO hits every machine in scope; demonstrate capability, don't push live code across the fleet.
- **Missing legacy GPP passwords.** They persist in SYSVOL from old configs and leak credentials to any domain user.
- **Forgetting GPO edit rights in tiering.** Whoever can edit a Tier 0 GPO is Tier 0, whatever their nominal role.

## References

- MITRE ATT&CK — T1484.001 (Group Policy Modification)
- Microsoft MS14-025 (Group Policy Preferences password vulnerability)
- BloodHound and PowerView GPO enumeration documentation
- Microsoft security baselines (delivered via GPO)
