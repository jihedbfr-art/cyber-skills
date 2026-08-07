---
name: privilege-escalation-chains
domain: 16-red-teaming-and-adversary-emulation
description: Use when emulating privilege escalation in an authorised engagement — chaining local escalations and misconfigurations into higher privilege, and how defenders break the chains.
difficulty: advanced
tags: [red-team, privilege-escalation, chains, authorized, emulation]
tools: []
---

## Purpose

Adversaries rarely land with the privileges they need — they escalate, often chaining several smaller weaknesses into a path from a low-privilege foothold to domain or system administrator. Emulating privilege escalation tests whether the organisation's hardening and detection break those chains. This skill covers the privilege-escalation phase of an authorised engagement conceptually — how escalations chain together — and, defensively, how each link is closed. It ties together the escalation-specific skills across the Linux, AD, and cloud domains.

## When to use it

During an authorised engagement (RoE) when escalating from a foothold. The defensive framing is central: escalation chains are made of individual, fixable weaknesses, and emulating them shows which links exist and which are broken.

## Procedure (authorised)

1. **Operate within the RoE.** Privilege escalation modifies systems and reaches sensitive access; stay in scope with deconfliction.
2. **Enumerate escalation opportunities** (the domain-specific skills do the detail):
   - **Local (host):** the Linux privilege-escalation and SUID/sudo skills, Windows local escalation — misconfigurations, weak permissions, vulnerable services.
   - **Active Directory:** Kerberoasting, AS-REP roasting, delegation abuse, ACL abuse, credential dumping (the AD domain) — the paths BloodHound maps.
   - **Cloud:** IAM privilege-escalation paths (the cloud domain).
3. **Understand escalation as a chain — the key concept.** A single weakness may only get you a little further; the escalation is usually a *sequence* — a low-priv foothold reads a credential, which accesses a service, which has a misconfiguration, which yields admin. Each link is a fixable weakness; the chain is the attack path. This is exactly what BloodHound visualises for AD.
4. **Emulate the actor's escalation approach** (the emulation-planning skill) and follow the chain toward the privilege objective.
5. **Focus on testing which links are broken — the defensive point.** The questions: does the hardening close the individual escalation opportunities (SUID cleaned, sudo tight, delegation removed, IAM scoped)? and does detection catch the escalation behaviour (credential dumping, roasting, IAM changes)? Every broken link stops the chain; every closed link is a defensive win.
6. **Understand how each link is closed** (the valuable defensive knowledge): the fix for each escalation is in its domain skill — SUID/sudo hardening (Linux), tiering/gMSA/removing delegation (AD), least-privilege IAM (cloud). Breaking any link in the chain stops the escalation, so the defensive strategy is closing the individual weaknesses that compose the paths.
7. **Report the escalation chain** — "the exact sequence of weaknesses that led to admin, and which links were closed or detected" is a highly actionable finding, because each link maps to a specific fix that breaks the whole path.

## Cheatsheet

```
adversaries escalate — chaining SMALL weaknesses into low-priv foothold -> admin
  emulating tests whether hardening + detection BREAK the chains

enumerate (domain skills do detail)
  LOCAL: Linux privesc (SUID/sudo), Windows local escalation — misconfig/weak perms/vuln services
  AD: Kerberoast, AS-REP, delegation, ACL abuse, credential dumping (BloodHound paths)
  CLOUD: IAM privilege-escalation paths

KEY CONCEPT: escalation = a CHAIN (foothold -> read cred -> access service -> misconfig -> admin)
  each link = a fixable weakness ; the chain = the attack path (BloodHound visualises this)

DEFENSIVE POINT: test which LINKS are BROKEN
  hardening closes individual opportunities (SUID clean, sudo tight, delegation removed, IAM scoped)
  detection catches escalation behaviour (dumping, roasting, IAM changes)
  BREAKING ANY LINK stops the chain -> strategy = close the individual weaknesses
report: exact sequence to admin + which links closed/detected (each link -> a specific fix)
```

## Reading the phase

- **A chain from low-priv foothold to admin with no links broken** = the escalation succeeded because every weakness in the path was present and undetected; a serious finding, but highly actionable — each link is a specific fix that would break the chain.
- **The escalation stopped because a link was closed** (hardened SUID, removed delegation, scoped IAM) = the defensive win; breaking any single link stops the whole chain, so a closed link is disproportionately valuable. Note which link held.
- **Escalation behaviour detected** (credential dumping, Kerberoasting, an IAM policy change) = the blue team caught the escalation; detection of these behaviours is a key defensive capability (the AD/detection domains).
- **A chain that maps cleanly to BloodHound-style paths** (for AD) = exactly what graph analysis shows; the report should express the chain as the sequence of edges, each a fixable permission/misconfiguration.
- **The exact escalation sequence and which links were closed/detected** = the most actionable red-team finding, because it turns "we got domain admin" into "here are the specific weaknesses to fix to break this path."

## Pitfalls

- **Operating outside RoE.** Escalation reaches sensitive access and modifies systems; stay in scope with deconfliction.
- **Reporting "we got admin" without the chain.** The value is the *sequence* of weaknesses and which links to close; a result without the chain isn't actionable. Document each link.
- **Focusing on the escalation over the defensive outcome.** The point is which links are broken and which escalation behaviours are detected; that's the finding.
- **Treating escalation as one weakness.** It's usually a chain; missing that leads to fixing one link while the path stays open through others. Map the whole chain.
- **Providing operational exploit tooling.** Conceptual by design; the detail and fixes are in the domain-specific skills. The value is the chain concept and the defensive strategy of breaking links.

## References

- MITRE ATT&CK — TA0004 (Privilege Escalation)
- The Linux (privilege-escalation, suid-sgid, sudo-hardening), Active Directory (kerberoasting, delegation, dcsync, tiering), and cloud (iam-privilege-escalation) domains — the links and their fixes
- BloodHound (chain/path visualisation) and the scoping/emulation-planning skills
- The detection-engineering and threat-hunting domains (escalation detection)
