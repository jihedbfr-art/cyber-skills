---
name: lateral-movement
domain: 16-red-teaming-and-adversary-emulation
description: Use when emulating lateral movement in an authorised engagement — how adversaries move between systems toward their objective, and how the blue team detects and constrains it.
difficulty: advanced
tags: [red-team, lateral-movement, authorized, active-directory, emulation]
tools: []
---

## Purpose

After the initial foothold, an adversary moves laterally toward their objective — the crown jewels are rarely where they land. Emulating lateral movement tests whether the organisation can detect and constrain an attacker moving through the network, which is often where there's the most opportunity to catch an intrusion. This skill covers the lateral-movement phase of an authorised engagement conceptually, and — the defensive core — how movement is detected and broken. It's the offensive counterpart to the threat-hunting lateral-movement skill.

## When to use it

The mid-engagement phase (post-foothold, pre-objective) of an authorised engagement under RoE. The defensive framing is central: lateral movement is detectable and constrainable, and emulating it tests both.

## Procedure (authorised)

1. **Operate within the RoE** and maintain deconfliction — lateral movement across production systems needs care and a way to distinguish your activity from a real attack.
2. **Understand the main movement techniques** (conceptual level) — how adversaries move:
   - **Remote execution** — executing on another host via SMB (PsExec-style), WMI, WinRM/PowerShell Remoting, RDP, or scheduled tasks.
   - **Credential-based movement** — using stolen/harvested credentials, pass-the-hash, or pass-the-ticket to authenticate to other systems (ties to the AD credential-dumping skill).
   - **Exploiting trust relationships** — AD delegation, trust relationships, and shared local admin credentials (the AD domain covers these paths).
3. **Emulate the actor's movement techniques** (the emulation-planning skill) — move the way the emulated adversary does, using the plan's scenario.
4. **Move toward the objective**, following the path from foothold toward the target systems (mirroring the BloodHound-style attack paths the AD domain maps).
5. **Focus on testing detection and constraint — the defensive point.** The questions: does the blue team detect the movement (unusual authentication, remote execution — the threat-hunting lateral-movement skill)? and does segmentation/tiering constrain it (can you actually reach the target, or do the AD tiering and network segmentation controls block the path)? Both are findings.
6. **Understand how movement is detected and broken** (the valuable defensive knowledge):
   - **Detection** — anomalous authentication patterns, remote-execution signatures, and credential-use anomalies (the threat-hunting and detection domains).
   - **Constraint** — network segmentation (limits reachability), AD tiered administration (breaks the credential-theft chain), and least privilege (limits what stolen credentials reach).
7. **Report the movement outcome** — "how far movement got, what was detected, and what constrained it" is a critical finding that drives the segmentation, tiering, and detection improvements that break attack paths.

## Cheatsheet

```
after foothold -> move laterally toward the objective (crown jewels rarely where you land)
  emulating it tests DETECT + CONSTRAIN of an attacker moving (often best chance to catch an intrusion)
  (conceptual ; authorised RoE + deconfliction)

techniques (conceptual)
  REMOTE EXECUTION: SMB/PsExec, WMI, WinRM/PS-Remoting, RDP, scheduled tasks
  CREDENTIAL-BASED: stolen creds, pass-the-hash/ticket (AD credential-dumping)
  TRUST RELATIONSHIPS: AD delegation, trusts, shared local admin (AD domain paths)

emulate the actor's movement -> move toward the objective (BloodHound-style paths)

DEFENSIVE POINT: test DETECTION + CONSTRAINT
  detect: anomalous auth, remote-exec signatures, credential-use anomalies [threat-hunting lateral-movement]
  constrain: network segmentation (reachability) | AD TIERING (breaks credential-theft chain) | least privilege
report: how far it got + what detected + what constrained -> drives segmentation/tiering/detection
```

## Reading the phase

- **Lateral movement reaching the objective undetected and unconstrained** = a serious finding on both axes; the blue team didn't catch the movement and the architecture didn't limit it. Drives both detection and segmentation/tiering improvements.
- **Movement detected via anomalous authentication or remote execution** = a defensive win; the blue team caught the attacker moving, which is often the best chance to catch an intrusion mid-flight (the threat-hunting lateral-movement skill).
- **Movement constrained by segmentation or AD tiering** (can't reach the target, or the credential-theft chain is broken) = the architecture doing its job; the path is blocked by design. A strong defensive outcome — note what constrained it.
- **Movement succeeding because of a flat network or shared local admin** = the architectural gaps that make movement easy; segmentation and unique local admin (LAPS) are the fixes.
- **Credential-based movement (pass-the-hash) reaching privileged systems** = the credential-theft chain working for the attacker; AD tiering is what breaks it.
- **How far movement got, detected, and constrained** = the critical finding; the phase's value is testing and improving the organisation's ability to detect and break attack paths.

## Pitfalls

- **Operating outside RoE / without deconfliction.** Lateral movement across production looks like a real attack and touches many systems; stay in scope with a deconfliction path.
- **Focusing on reaching the objective over the defensive outcome.** The value is whether movement is detected and constrained; both are the findings, not just whether you got there.
- **Testing only detection, not constraint (or vice versa).** Lateral movement tests both the blue team's detection *and* the architecture's segmentation/tiering; assess both.
- **Not emulating the actor's techniques.** Move the way the emulated adversary does, per the plan, to test realistic detection.
- **Providing operational tooling.** Conceptual by design; the value is understanding movement and its detection/constraint, not ready-to-fire lateral-movement tools.

## References

- MITRE ATT&CK — TA0008 (Lateral Movement), T1021, T1550
- The threat-hunting lateral-movement-hunting skill and the Active Directory domain (ntlm-relay, dcsync, tiered-admin-model, the paths)
- The network-segmentation skill and scoping/emulation-planning skills
- BloodHound (attack-path mapping — both sides use it)
