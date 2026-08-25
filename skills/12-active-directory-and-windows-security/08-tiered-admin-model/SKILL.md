---
format: "v2"
name: "tiered-admin-model"
title: "Tiered Admin Model"
title_fr: "Modèle d'administration par niveaux"
description: "Use when designing AD administration to break credential-theft attack paths — the tiered model that stops a phished laptop from becoming Domain Admin."
description_fr: "À utiliser pour concevoir l'administration AD de façon à casser les chemins de vol d'identifiants — le modèle par niveaux (tiering) qui empêche un poste phishé de mener jusqu'à Domain Admin."
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

Most AD compromises follow the same shape: an attacker phishes a workstation, dumps credentials from memory, finds an admin who logged into that workstation, steals *their* credentials, and climbs. The tiered administration model breaks that chain by rule: privileged credentials are never exposed on lower-trust systems, so there's no admin token on the phished laptop to steal. This skill covers designing that model — the single most effective structural defence against AD credential-theft escalation.

### When to use it

Designing or hardening AD administration, or after an assessment (BloodHound) shows short paths from workstations to Domain Admins that run through admin sessions on low-trust machines. It's the architectural fix behind many of the specific findings in the credential-dumping, delegation, and enumeration skills.

### The model

Split administration into tiers by what a credential controls, and forbid credentials from crossing downward:

- **Tier 0** — the keys to the kingdom: domain controllers, AD itself, and anything that can control identity (PKI, some management systems). Compromise here = domain compromise.
- **Tier 1** — servers and applications (the workloads).
- **Tier 2** — user workstations and devices.

**The core rule: a higher-tier credential is never used on a lower-tier system.** A Domain Admin (Tier 0) never logs into a server (Tier 1) or a workstation (Tier 2). Because the credential is never present there, an attacker who owns a workstation can't steal a Tier 0 token from it. Admins use separate accounts per tier, and administer each tier only from systems of that tier (privileged access workstations for Tier 0).

### Procedure

1. **Classify assets into tiers** — identify Tier 0 precisely (DCs, AD, PKI, and anything that can gain control of them; this last part is often missed and is where tiering leaks). Then Tier 1 servers and Tier 2 workstations.
2. **Separate admin accounts per tier.** A person who administers all three uses three distinct accounts, and each account is only ever used on its own tier's systems. No shared "does everything" admin account.
3. **Enforce the no-downward-exposure rule technically**, not just by policy: use logon restrictions (deny Tier 0 accounts the right to log on to Tier 1/2 systems), so a mistake can't expose a high-tier credential on a low-tier box.
4. **Use Privileged Access Workstations (PAWs)** for Tier 0 administration — hardened, dedicated machines used only for admin, never for email/web (where the phish lands).
5. **Clean up existing violations** — the model only works if there are no current paths that break it. Use BloodHound to find where high-tier credentials are (or have been) exposed on lower-tier systems, and remediate those sessions and rights.
6. **Protect the enablers** — GPO edit rights, delegation, and ACLs that reach Tier 0 must themselves be Tier 0 (whoever can edit a DC-linked GPO is effectively Tier 0). Tiering has to include the indirect control paths, not just direct logons.

### Cheatsheet

```
tiers (by what the credential controls)
  Tier 0  DCs, AD, PKI, + anything that can CONTROL them  (= domain if breached)
  Tier 1  servers / applications
  Tier 2  user workstations / devices

THE RULE: a higher-tier credential is NEVER used on a lower-tier system.
  -> Domain Admin never logs into a server or workstation
  -> separate admin account per tier; administer each tier from its own tier
  -> Tier 0 admin only from a hardened PAW (no email/web on it)

why it works: the phished workstation has no Tier 0 token to steal ->
              the credential-theft escalation chain is broken by design

don't forget indirect Tier 0: GPO edit rights / delegation / ACLs that
  reach a DC ARE Tier 0, whoever holds them.
```

### Reading an environment

- **Domain Admins logging into workstations/servers** = the core rule is broken; every such session is a Tier 0 token sitting on a lower-trust machine, waiting to be dumped. The finding that tiering exists to eliminate.
- **Shared admin accounts used across tiers** = one stolen credential crosses tier boundaries; defeats the model. Separate per tier.
- **Short BloodHound paths from a workstation to Domain Admins** = tiering isn't enforced; the path usually runs through an admin session or right that shouldn't exist at that tier.
- **Indirect Tier 0 held by lower tiers** (GPO edit rights, delegation, ACLs reaching DCs) = a hidden tier violation; the model must include these control paths, not just interactive logons.
- **Enforced logon restrictions + PAWs + no cross-tier sessions** = the model working; credential theft on a workstation no longer escalates.

### Pitfalls

- **Policy without technical enforcement.** "Admins shouldn't log into workstations" as a guideline gets violated. Enforce it with logon restrictions so the credential physically can't land there.
- **Missing indirect Tier 0.** Focusing on direct DC logons while ignoring GPO/delegation/ACL paths that control DCs — those holders are Tier 0 too, and attackers use exactly those paths.
- **Not cleaning up existing exposure.** Deploying the model forward while old admin sessions and rights still create paths leaves the escalation intact. Remediate the current graph.
- **Shared admin accounts.** One account used across tiers is a single credential that crosses every boundary. One account per tier.
- **Treating it as all-or-nothing.** Even partial tiering (protecting Tier 0 first) meaningfully raises the bar; scope it, but start with the domain controllers.

### References

- Microsoft — Securing Privileged Access / Enterprise Access Model (the tiered model and PAWs)
- BloodHound (to find and verify tier violations)
- MITRE ATT&CK — Credential Access and Lateral Movement tactics (what tiering breaks)
- Microsoft Protected Users group and authentication policies

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.