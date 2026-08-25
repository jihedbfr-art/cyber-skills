---
format: "v2"
name: "network-access-control"
title: "Network Access Control"
title_fr: "Contrôle d'accès réseau"
description: "Use when controlling which devices are allowed onto the network — 802.1X and NAC — so an unauthorised device can't just plug in and join, and posture is checked before access."
description_fr: "À utiliser pour contrôler quels équipements sont autorisés à rejoindre le réseau — 802.1X et NAC — afin qu'un appareil non autorisé ne puisse pas simplement se brancher et se connecter, l'état de conformité étant vérifié avant tout accès."
domain: "02-network-security"
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

On many networks, anyone who plugs into a port or joins the Wi-Fi is on the network — no questions asked. That means a rogue laptop, a compromised IoT device, or an attacker in the building gets network access for free. Network Access Control (NAC), built on 802.1X, changes that: devices must authenticate before they're allowed on, and can be checked for security posture and placed in the right segment. This skill covers using NAC to make network access a controlled decision rather than a default.

### When to use it

Hardening physical and wireless network access, especially in offices with many ports and devices, or after an assessment showed that plugging in grants immediate access. It's the enforcement layer that makes segmentation and the "only authorised devices" principle real at the point of connection.

### How it works

- **802.1X** is the standard for port-based access control. A connecting device (supplicant) authenticates to a switch/AP (authenticator), which checks the credentials against a RADIUS server. Until authentication succeeds, the port grants no network access (or only a restricted quarantine).
- **NAC** builds on this: beyond "who are you", it can check "what state are you in" (posture — patched, AV running, managed device) and dynamically assign the device to a network segment/VLAN based on identity and posture.

### Procedure

1. **Decide the policy** — what should happen when a device connects: authorised managed device → its proper segment; unknown/BYOD → restricted or guest segment; failed/unmanaged → quarantine or denied. NAC is only as good as this policy.
2. **Deploy 802.1X** on switches and wireless with a RADIUS server as the authentication backend. Devices authenticate (via certificates for managed devices — stronger — or credentials) before getting access:
   ```
   # RADIUS (e.g. FreeRADIUS) as the 802.1X auth backend;
   # switches/APs configured as authenticators pointing at it
   ```
3. **Handle devices that can't do 802.1X** — printers, IoT, cameras often can't authenticate. Use MAB (MAC Authentication Bypass) for these, but understand MAC addresses are spoofable, so MAB devices go in a tightly-restricted segment, not the trusted network.
4. **Add posture checks** where the NAC supports it — verify a connecting device is managed, patched, and running required security software before granting full access; non-compliant devices get remediation/quarantine access only.
5. **Assign segments dynamically** — NAC can drop each device into the right VLAN by identity/posture, tying access control to segmentation (the segmentation skill) automatically.
6. **Plan the rollout carefully** — 802.1X misconfigured locks legitimate devices off the network. Roll out in monitor mode first (authenticate and log, but don't enforce), fix what breaks, then enforce.

### Cheatsheet

```
802.1X flow
  device (supplicant) -> switch/AP (authenticator) -> RADIUS server
  no access until authenticated (or restricted quarantine VLAN)

policy by identity + posture
  managed + compliant   -> proper segment
  BYOD / unknown        -> restricted / guest
  non-compliant / failed -> quarantine / remediation only

device types
  managed devices  -> 802.1X with CERTIFICATES (strongest)
  can't do 802.1X (printers/IoT/cameras) -> MAB (MAC bypass)
      but MACs are SPOOFABLE -> MAB devices in a tight restricted segment

rollout: MONITOR mode first (log, don't enforce) -> fix -> then enforce
  (misconfigured 802.1X locks out legit devices)
```

### Reading the environment

- **Any device that plugs in gets network access** = no access control at the connection point; a rogue or compromised device joins freely. NAC/802.1X is the fix, and this is the gap it closes.
- **802.1X on managed devices with certificates + dynamic segment assignment** = the strong state; access is authenticated and devices land in the right zone automatically.
- **MAB devices (printers/IoT) on the trusted network** = a spoofable-MAC path onto trusted segments; move them to a restricted zone since MAB isn't real authentication.
- **No posture checking** = authenticated but unpatched/compromised devices still get full access; posture checks add the "what state are you in" layer.
- **802.1X enforced without a monitor-mode rollout** = high risk of locking out legitimate devices and causing an outage; stage it.
- **Guest/BYOD isolated, managed devices authenticated, IoT restricted** = access control working end to end.

### The fix / best practice

- **Deploy 802.1X** on wired and wireless with RADIUS, using **certificate-based auth for managed devices** (stronger and phishing-resistant vs credentials).
- **Define access policy by identity and posture** — the right segment for managed/compliant, restriction for BYOD/unknown, quarantine for non-compliant.
- **Restrict MAB devices** (printers, IoT that can't do 802.1X) to tightly-scoped segments, treating MAC-based access as weak.
- **Add posture assessment** so device health, not just identity, gates full access.
- **Tie NAC to segmentation** with dynamic VLAN assignment, so access control and network zoning work together.
- **Roll out in monitor mode first**, then enforce, to avoid locking out legitimate devices.

### Pitfalls

- **Rolling out enforcement without monitor mode.** Misconfigured 802.1X locks legitimate devices off the network — a self-inflicted outage. Log-only first, fix, then enforce.
- **Trusting MAB as authentication.** MAC addresses are trivially spoofed; MAB is a compatibility fallback, not security. Restrict those devices tightly.
- **Authenticating identity but ignoring posture.** A legitimate but compromised/unpatched device still gets in; posture checks catch that.
- **NAC without segmentation.** Controlling who gets on is half the value; putting each device in the right zone (dynamic VLAN) is the other half.
- **Forgetting the un-authenticatable devices.** Printers, cameras, and IoT need a planned path (restricted segment), or NAC breaks them or gets bypassed.

### References

- IEEE 802.1X standard and RADIUS (FreeRADIUS documentation)
- NIST guidance on network access control
- The network-segmentation and MFA (certificate auth) skills
- CWE-284 (improper access control)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.