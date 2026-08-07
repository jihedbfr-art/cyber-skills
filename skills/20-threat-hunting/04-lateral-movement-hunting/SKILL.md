---
name: lateral-movement-hunting
domain: 20-threat-hunting
description: Use when hunting for an attacker moving between hosts — the authentication patterns, remote-execution, and access anomalies that reveal lateral movement across the network.
difficulty: intermediate
tags: [threat-hunting, lateral-movement, authentication, network]
tools: []
---

## Purpose

An attacker rarely lands where they want to end up — they gain a foothold and move laterally toward the crown jewels. Hunting for lateral movement catches the intrusion in its middle phase, after initial access but before the objective, where there's often the most opportunity to detect it. This skill covers finding the authentication, remote-execution, and access patterns that betray movement between hosts, one of the most valuable hunts because lateral movement is unavoidable for most attacker goals.

## When to use it

Hunting for active or past intrusions, especially in environments where an attacker who gains one foothold could reach sensitive systems (which is most). It pairs with the AD domain (which covers the specific techniques) and network hunting.

## Procedure

1. **Know what normal movement looks like — baseline first.** Lateral movement hunting is anomaly-heavy: you're looking for authentication and access that deviates from normal patterns. Establish who normally logs into what, from where — admins to servers, users to their own workstations. The anomaly is movement that doesn't fit (the anomaly-baselining skill).
2. **Hunt authentication patterns.** The core signal: unusual logons — a workstation account authenticating to servers, a user logging into many hosts in a short time, service accounts logging on interactively, logons from unexpected sources, or network logons (type 3) between hosts that don't normally talk. In AD, event 4624/4625 with logon types and source hosts is the data.
3. **Hunt remote execution.** Attackers move by executing on remote hosts — via SMB (PsExec-style), WMI, WinRM/PowerShell Remoting, RDP, or scheduled tasks. Hunt for these remote-execution signatures, especially from unusual sources or to sensitive targets.
4. **Hunt credential-use anomalies.** Lateral movement uses stolen credentials — pass-the-hash/ticket, a credential used from a host it's never been used from, an admin credential appearing on a workstation. Combine with the AD credential-dumping context.
5. **Follow the graph.** Lateral movement is a *sequence* across hosts; a single unusual logon is a data point, a *chain* (host A → B → C in sequence, converging toward sensitive systems) is the movement. Correlate across hosts and time to see the path.
6. **Prioritise movement toward value.** Movement heading for domain controllers, databases, or sensitive systems is far more concerning than movement between low-value hosts. Weight by destination.
7. **Investigate and escalate** — a confirmed lateral-movement chain is an active intrusion; escalate to IR, and the pattern feeds detection and the AD hardening/tiering work (which breaks these paths).

## Cheatsheet

```
attacker: foothold -> moves laterally -> crown jewels. hunt the MIDDLE phase.

1. BASELINE normal movement (who logs into what, from where) — anomaly-heavy hunt
2. AUTHENTICATION anomalies (the core signal)
     workstation acct -> servers | user -> many hosts fast | service acct interactive
     | logon from unexpected source | network logons (type 3) between odd host pairs
     (AD: 4624/4625 + logon type + source host)
3. REMOTE EXECUTION: SMB/PsExec, WMI, WinRM/PS-Remoting, RDP, scheduled tasks
     -> from unusual source / to sensitive target
4. CREDENTIAL anomalies: pass-the-hash/ticket, cred used from a new host, admin on workstation
5. FOLLOW THE GRAPH: single odd logon = data point ; CHAIN (A->B->C toward sensitive) = movement
6. PRIORITISE movement toward VALUE (DC/DB/sensitive > low-value hosts)
7. confirmed chain = active intrusion -> IR ; feeds detection + AD tiering (breaks the paths)
```

## Reading the hunt

- **A chain of logons across hosts converging on sensitive systems** = the lateral-movement signature; the *sequence* toward value is the finding, far more than any single logon. This is what the hunt exists to catch — an active intrusion mid-flight.
- **A workstation account authenticating to servers, or a user logging into many hosts rapidly** = anomalous authentication that doesn't fit normal patterns; strong movement indicators, but require the baseline to recognise as abnormal.
- **Remote execution (PsExec/WMI/WinRM) from an unusual source to a sensitive host** = a movement mechanism in use; high-value when it targets DCs or crown jewels.
- **A credential appearing on a host it's never been used from** (especially admin on a workstation) = likely stolen-credential reuse for movement; combine with the auth anomaly.
- **Movement between low-value hosts** = lower priority than movement toward sensitive targets; weight by destination.
- **An isolated unusual logon with no chain** = possibly benign (an admin doing legitimate work); the chain and the direction-toward-value are what separate real movement from noise.

## Pitfalls

- **No baseline.** Lateral-movement hunting is anomaly detection; without knowing normal authentication and access patterns, you can't spot the abnormal. Baseline first.
- **Hunting single events, not chains.** One unusual logon is often benign; the *sequence* across hosts toward sensitive systems is the movement. Correlate the graph.
- **Ignoring direction.** Movement toward crown jewels is the concern; treating all movement equally buries the important paths. Weight by destination value.
- **Missing the remote-execution and credential angles.** Authentication is one signal; remote execution (WMI/WinRM/PsExec) and credential-use anomalies complete the picture. Hunt all three.
- **Alert fatigue from legitimate admin activity.** Admins move between hosts legitimately; baselining and context (is this admin's normal pattern?) separate them from attackers.

## References

- The AD ad-enumeration-bloodhound, dcsync, ntlm-relay, and tiered-admin-model skills
- MITRE ATT&CK — TA0008 (Lateral Movement), T1021 (Remote Services), T1550 (Use Alternate Auth Material)
- The anomaly-baselining, endpoint-hunting, and hypothesis-driven-hunting skills
- SANS threat hunting resources
