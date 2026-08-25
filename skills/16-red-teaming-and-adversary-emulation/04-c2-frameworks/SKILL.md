---
format: "v2"
name: "c2-frameworks"
title: "C2 Frameworks"
title_fr: "Frameworks de commande et contrôle (C2)"
description: "Use when operating command-and-control in an authorised engagement — how C2 frameworks work, using them within scope, and how the blue team detects C2 traffic and beaconing."
description_fr: "À utiliser pour opérer un canal de commande et contrôle (C2) dans un engagement autorisé — le fonctionnement des frameworks C2, leur usage dans le périmètre autorisé, et comment l'équipe défensive détecte le trafic C2 et le beaconing."
domain: "16-red-teaming-and-adversary-emulation"
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

Command-and-control (C2) is how an operator communicates with compromised systems during an engagement — issuing commands and receiving results. C2 frameworks (Cobalt Strike, Mythic, Sliver, and others) provide this infrastructure. This skill covers understanding and operating C2 in an authorised engagement at the conceptual level, and — the defensive core — how C2 is detected, so the engagement tests the blue team's ability to catch an operator's communications. It stays conceptual, consistent with the repo's scope.

### When to use it

The post-foothold phase of an authorised engagement (with a signed RoE), when maintaining access and operating on compromised systems. The defensive framing is central: C2 detection is a major blue-team capability, and emulating C2 tests it directly.

### Procedure (authorised)

1. **Operate strictly within the RoE**, using only approved C2 infrastructure and techniques, with deconfliction so the blue team can distinguish your activity from a real intrusion if needed.
2. **Understand what C2 frameworks provide** — a server/operator interface, an implant/agent on the compromised host, and a communication channel between them. The implant beacons back to the C2 server for commands. Frameworks offer various channels (HTTP/HTTPS, DNS, others) and evasion features.
3. **Emulate the C2 the actor uses** (the emulation-planning skill) — match the C2 profile/channel to the emulated adversary's known tradecraft, so you're testing detection of *that actor's* communications.
4. **Focus on testing C2 detection — the defensive point.** The key question isn't "does the C2 work" but "does the blue team detect it?". C2 detection is a core capability, and the engagement tests it: is the beaconing caught (the beaconing-detection skill)? the C2 domains/IPs? the traffic patterns? Document what was and wasn't detected.
5. **Understand how C2 is detected** (the defensive knowledge that makes the emulation valuable):
   - **Beaconing** — regular-interval callbacks are a strong signature (the threat-hunting beaconing skill), even with jitter and over HTTPS.
   - **Network indicators** — C2 domains/IPs (often new/rare), TLS/JA3 fingerprints, and unusual destinations (the DNS/proxy hunting skill).
   - **Endpoint** — the implant's behaviour (injection, unusual processes, the malware C2-analysis skill).
   - **Traffic anomalies** — data patterns, volumes, and protocol misuse.
6. **Vary tradecraft to test detection depth** — different channels (DNS vs HTTPS), jitter, and evasion test whether the blue team catches C2 only in the obvious case or in the evasive one too.
7. **Report the C2 detection outcome** — for the blue team, "was our C2 detected, and how could detection improve" is a high-value finding, feeding the beaconing/network detection skills.

### Cheatsheet

```
C2 = operator <-> compromised systems (issue commands, get results)
  frameworks (Cobalt Strike/Mythic/Sliver): server + implant/agent + channel ; implant BEACONS back
  (conceptual ; authorised RoE ; approved infra + deconfliction)

emulate the actor's C2 (channel/profile matching their tradecraft)

DEFENSIVE POINT: test C2 DETECTION (does blue team catch it?), not "does it work"
how C2 is detected (the valuable knowledge)
  BEACONING: regular-interval callbacks = strong signature (even jitter + HTTPS) [beaconing-detection]
  NETWORK: C2 domains/IPs (new/rare), TLS/JA3 fingerprint, unusual destinations [dns/proxy hunting]
  ENDPOINT: implant behaviour (injection, unusual processes) [malware c2-analysis]
  TRAFFIC anomalies: data patterns, volumes, protocol misuse

VARY tradecraft (DNS vs HTTPS, jitter, evasion) -> test detection DEPTH
report the detection outcome -> improve beaconing/network detection
```

### Reading the phase

- **C2 beaconing not detected** = a detection gap; regular-interval callbacks are a strong, catchable signature (even with jitter and over HTTPS), so missing them points to a beaconing-detection gap the blue team should close. A high-value finding.
- **C2 detected via beaconing/network indicators** = a defensive win; the blue team caught the operator's communications, which is exactly what the engagement tests. Note how it was caught.
- **Evasive C2 (DNS channel, high jitter) evading detection while obvious C2 is caught** = the detection is shallow — it catches the easy case but not the evasive one. Varying tradecraft reveals detection depth.
- **The C2 domains/IPs being new/rare and uncategorised** = exactly the network indicators detection should surface (the DNS/proxy and threat-intel skills); test whether the blue team flags them.
- **Endpoint detection of the implant** (injection, unusual process) = the host-based catch; C2 has both network and endpoint detection angles.
- **The C2 detection outcome documented** = the engagement's defensive value; "was our C2 caught and how to improve detection" drives the blue team's C2-detection capability.

### Pitfalls

- **Operating outside the RoE or without deconfliction.** C2 emulation looks exactly like a real intrusion; use approved infrastructure within scope, with a way to deconflict from a genuine attack.
- **Focusing on C2 functionality over detection.** The value is testing whether the blue team catches the C2, not whether the framework works. The detection outcome is the finding.
- **Only testing obvious C2.** If you only run default HTTPS C2, you test the easy case; vary channels, jitter, and evasion to test detection depth.
- **Not emulating the actor's C2.** Match the C2 tradecraft to the emulated adversary so you test detection of realistic communications.
- **Providing operational C2 tooling/configs.** This skill stays conceptual; the value is understanding C2 and its detection, not ready-to-use offensive infrastructure.

### References

- MITRE ATT&CK — TA0011 (Command and Control), T1071, T1573
- The threat-hunting beaconing-detection and dns-and-proxy-hunting skills, malware c2-and-network-analysis skill
- The scoping-and-rules-of-engagement and attack-emulation-planning skills
- MITRE Caldera (open emulation) and C2 detection research (the defensive references)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.