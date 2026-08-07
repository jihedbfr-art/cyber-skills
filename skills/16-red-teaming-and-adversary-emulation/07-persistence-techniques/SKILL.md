---
name: persistence-techniques
domain: 16-red-teaming-and-adversary-emulation
description: Use when emulating persistence in an authorised engagement — how adversaries maintain access across reboots and remediation, and how defenders detect and hunt the footholds.
difficulty: advanced
tags: [red-team, persistence, authorized, emulation, detection]
tools: []
---

## Purpose

Persistence is how an adversary keeps access — surviving reboots, credential changes, and even partial remediation. Emulating persistence tests whether the organisation can find the footholds an attacker leaves, which is exactly what incident-response eradication must do. This skill covers the persistence phase of an authorised engagement conceptually — the surfaces adversaries use — and, defensively, how each is detected and hunted. It's the offensive counterpart to the IR eradication and threat-hunting skills.

## When to use it

During an authorised engagement (RoE) when establishing durable access, and it directly informs the blue team's ability to find and remove footholds. The defensive framing is central: persistence mechanisms are detectable and huntable, and emulating them tests that.

## Procedure (authorised)

1. **Operate within the RoE** — persistence modifies systems (registry, services, tasks, accounts); stay in scope, document what you establish, and clean it up afterward (the RoE should require removing all persistence at engagement end).
2. **Understand the persistence surfaces** (conceptual) — where adversaries establish footholds:
   - **Windows:** registry Run keys, services, scheduled tasks, WMI event subscriptions, startup folders, and account manipulation.
   - **Linux:** cron jobs, systemd units/timers, `~/.ssh/authorized_keys`, rc/profile scripts, and startup services.
   - **Active Directory:** rogue accounts, delegation, Golden/Silver Tickets, ACL backdoors, and DCSync rights (the AD domain).
   - **Cloud:** rogue IAM users/keys, modified trust policies, Lambda backdoors (the cloud domain).
3. **Emulate the actor's persistence** (the emulation-planning skill) — establish footholds the way the emulated adversary does.
4. **Establish multiple footholds** if realistic — real adversaries plant several, so that removing one doesn't evict them. This tests whether the blue team finds *all* of them (which eradication requires).
5. **Focus on testing detection and hunting — the defensive point.** The questions: does the blue team detect the persistence being established (a new service, a Run key, an added account)? and can they *find* the footholds when hunting (the threat-hunting endpoint skill) and eradicate them (the IR eradication skill)? Persistence that's established undetected and can't be found is the finding.
6. **Understand how persistence is detected and hunted** (the valuable defensive knowledge): the surfaces change rarely, so anomalies stand out — a new Run key/service/task, an added SSH key, a rogue account are all detectable and huntable (the detection and threat-hunting endpoint/persistence skills). The blue team must hunt all these surfaces to eradicate an intrusion.
7. **Report the persistence outcome** — "which footholds were established, whether they were detected, and whether the blue team found them all" is a critical finding that directly tests and improves eradication capability.

## Cheatsheet

```
persistence = keep access across reboots / credential changes / partial remediation
  emulating tests whether the org can FIND the footholds (= what IR eradication must do)
  (conceptual ; authorised RoE ; DOCUMENT + CLEAN UP all persistence at end)

surfaces (where footholds go)
  WINDOWS: Run keys | services | scheduled tasks | WMI subscriptions | startup | account manipulation
  LINUX: cron | systemd units/timers | authorized_keys | rc/profile | startup services
  AD: rogue accounts | delegation | Golden/Silver tickets | ACL backdoors | DCSync rights
  CLOUD: rogue IAM users/keys | modified trust policies | Lambda backdoors

emulate the actor's persistence -> establish MULTIPLE footholds (real adversaries do -> removing one != evicted)
  -> tests whether blue team finds ALL of them (eradication requires it)

DEFENSIVE POINT: test DETECTION (new service/Run key/account caught?) + HUNTING (can they FIND them?)
  surfaces change rarely -> anomalies stand out (detectable + huntable)
report: footholds established + detected? + found all? -> tests/improves eradication
```

## Reading the phase

- **Persistence established undetected and not found when the blue team looks** = the core finding; if footholds go unnoticed and can't be hunted down, an intrusion can't be fully eradicated. Drives the detection and hunting of persistence surfaces.
- **Persistence detected when established** (a new service/Run key/account alerted) = a defensive win; these surfaces change rarely, so an anomaly should stand out. Note what was caught.
- **Multiple footholds established, blue team finding only some** = an eradication gap; removing some while others remain means the attacker survives (the IR eradication skill's warning). Real adversaries plant several precisely for this — testing it is valuable.
- **A rogue AD account or DCSync backdoor** = high-value persistence; the AD credential-dumping/tiering skills cover finding and removing these. Test whether the blue team catches them.
- **The footholds mapping to well-known persistence surfaces** (Run keys, cron, authorized_keys) = exactly what hunting and detection should cover; the report should list each surface and whether it was detected/hunted.
- **Which footholds established, detected, and found** = the critical finding; the phase's value is testing and improving the organisation's ability to find and eradicate persistence.

## Pitfalls

- **Not cleaning up persistence.** Leaving real footholds after the engagement is dangerous; the RoE must require removing all persistence, and you must document and remove it.
- **Establishing one foothold.** Real adversaries plant several so removing one doesn't evict them; testing only single persistence understates the eradication challenge. Establish multiple where realistic.
- **Focusing on establishing persistence over the defensive outcome.** The value is whether it's detected and can be found; that's the finding, not the persistence itself.
- **Missing the eradication angle.** Persistence directly tests whether the blue team can *fully* eradicate an intrusion (find all footholds); frame the findings that way.
- **Providing operational persistence tooling.** Conceptual by design; the surfaces and their detection are the value, with detail in the domain skills.

## References

- MITRE ATT&CK — TA0003 (Persistence): T1547, T1053, T1543, T1546, T1098, etc.
- The incident-response eradication-and-recovery skill, threat-hunting endpoint-hunting skill, and detection-engineering domain
- The Active Directory (dcsync, delegation, tiering) and cloud (incident-response) domains
- The scoping and emulation-planning skills
