---
format: "v2"
name: "eradication-and-recovery"
title: "Eradication And Recovery"
title_fr: "Éradication et remise en service"
description: "Use after containment — removing the attacker's foothold completely and restoring to clean, trustworthy operation without reintroducing the compromise or leaving persistence behind."
description_fr: "À utiliser après le confinement, pour éliminer complètement l'accès de l'attaquant et revenir à un fonctionnement sain et fiable, sans réintroduire la compromission ni laisser de persistance en place."
domain: "22-incident-response"
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

Containment stops the spread; eradication removes the attacker; recovery gets you back to normal. The trap in this phase is declaring victory too early — cleaning the obvious malware while missing the persistence mechanism or the entry vector, so the attacker walks back in a week later. This skill covers eradicating completely and recovering to a state you can actually trust.

### When to use it

After the incident is contained and you understand the scope (the containment and evidence skills come first). Don't start eradication until you know how the attacker got in and what they touched — cleaning before you understand the intrusion just tips them off and leaves the door open.

### Procedure

1. **Understand the full scope first.** Before removing anything, be confident you know the entry vector, every affected system, and every persistence mechanism. Eradicating what you can see while missing a hidden foothold is the classic failure — the investigation (forensics, threat hunting) informs this.
2. **Close the entry vector.** Patch the exploited vulnerability, disable the abused account, fix the misconfiguration. If you don't, you'll re-compromise as soon as you restore. This is the step teams skip in a hurry.
3. **Remove persistence, not just payloads.** Attackers plant multiple footholds — scheduled tasks, services, startup entries, web shells, rogue accounts, modified binaries, cloud IAM changes, mail rules. Hunt for all of them; killing one malware process while a backdoor account remains achieves nothing.
4. **Rebuild rather than clean, for anything seriously compromised.** For a host with deep compromise, rebuilding from a known-good image is more trustworthy than trying to disinfect — you can rarely prove a "cleaned" box is truly clean. Restore data from backups verified to predate the compromise.
5. **Reset credentials broadly.** Assume anything the attacker could reach is compromised — passwords, keys, tokens, and secrets on affected systems. In an AD compromise, this includes the domain (krbtgt twice) as covered in the credential-dumping skill.
6. **Recover in a controlled order**, validating each system is clean and functioning before returning it to production, and monitor closely afterward.
7. **Watch for reinfection.** Heightened monitoring on the recovered environment — attackers often try to return, and a re-compromise means eradication missed something.

### Cheatsheet

```
sequence (don't skip step 1)
  1. scope understood?  entry vector + all hosts + all persistence known
  2. close the ENTRY VECTOR (patch/disable/fix) — or you re-compromise
  3. remove ALL persistence, not just the visible payload:
       scheduled tasks, services, startup, web shells, rogue accounts,
       modified binaries, cloud IAM changes, mail-forward rules
  4. rebuild deeply-compromised hosts from known-good (don't "clean")
       restore data from backups predating the compromise
  5. reset credentials broadly (assume all reachable secrets burned;
       AD -> rotate krbtgt twice)
  6. recover in controlled order, validate each host before prod
  7. heightened monitoring — watch for reinfection

golden rule: eradicate what you UNDERSTAND, not just what you SEE.
```

### Reading the situation

- **Uncertainty about the entry vector** = not ready to eradicate. Restoring without closing the door guarantees re-compromise. Investigate first.
- **A single obvious malware found** = keep looking; real intrusions have multiple persistence mechanisms. One web shell rarely travels alone.
- **A deeply compromised host** = rebuild, don't clean — you can't prove disinfection worked on a box the attacker had root on.
- **Backups near the compromise date** = verify they predate the intrusion and aren't backdoored before trusting them (ties into ransomware-response).
- **Reinfection after recovery** = eradication missed a foothold or the entry vector; go back to scoping, don't just re-clean.

### Pitfalls

- **Eradicating before understanding.** Cleaning what you see while missing hidden persistence tips off the attacker and leaves them in. Scope first.
- **Skipping the entry vector.** The most common re-compromise cause — you remove the malware but leave the unpatched hole it came through.
- **Cleaning instead of rebuilding.** A "disinfected" deeply-compromised host can't be trusted; rebuild from known-good.
- **Narrow credential reset.** Resetting only the obviously-affected account while the attacker harvested others leaves valid keys behind.
- **Declaring done at recovery.** Without heightened post-recovery monitoring, a returning attacker goes unnoticed until it's a new incident.

### References

- NIST SP 800-61r2 (Containment, Eradication, and Recovery)
- SANS Incident Handler's Handbook
- CISA eradication and recovery guidance
- MITRE ATT&CK — Persistence tactic (what to hunt during eradication)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.