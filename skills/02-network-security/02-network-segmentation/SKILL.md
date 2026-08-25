---
format: "v2"
name: "network-segmentation"
title: "Network Segmentation"
title_fr: "Segmentation réseau"
description: "Use when designing or reviewing network segmentation — dividing a network so a foothold in one zone can't reach everything, and verifying the boundaries actually hold."
description_fr: "À utiliser pour concevoir ou auditer la segmentation réseau — diviser le réseau en zones pour qu'une compromission dans l'une ne permette pas d'atteindre tout le reste — et vérifier que les frontières tiennent réellement leurs promesses."
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

A flat network is a single blast radius: phish one laptop and the attacker can reach the databases, the domain controllers, everything. Segmentation divides the network into zones with controlled boundaries between them, so a compromise in one zone doesn't become a compromise of all. This skill covers designing segmentation that limits lateral movement, and — the part that gets skipped — verifying the boundaries do what the diagram claims.

### When to use it

Designing a network, reviewing one after an assessment showed easy lateral movement, or planning to contain a known-flat environment. It's the architectural counterpart to the lateral-movement techniques in the red-team domain — segmentation is what makes those hard.

### Procedure

1. **Group assets by trust and function.** Identify what belongs together and what must be kept apart: user workstations, servers, databases, management/admin, DMZ (internet-facing), OT/IoT, and the crown jewels. Sensitive systems get their own segments.
2. **Define the allowed flows between zones — default deny.** Segmentation is only real if traffic between zones is restricted to what's explicitly needed. Workstations reach the app tier on specific ports; the app tier reaches the database tier; the database tier initiates nothing outbound. Everything not allowed is denied.
3. **Isolate the highest-value and highest-risk zones hardest.** Databases and domain controllers (Tier 0 — see the AD tiering skill) should be reachable only from narrow, defined sources. OT/IoT and guest networks should be walled off from the corporate network entirely.
4. **Enforce with the right mechanism at the right layer** — VLANs plus firewall/ACL rules between them, host firewalls, cloud security groups, or microsegmentation. The mechanism matters less than the enforced default-deny between zones.
5. **Verify the boundaries actually hold** — this is where segmentation projects fail. From a host in one zone, test what you can actually reach in another; a VLAN without an enforcing ACL is not segmentation:
   ```
   # from a workstation-zone host, what's reachable in the DB zone?
   nmap -sS -p- <db-zone-range>       # should be denied / minimal, not open
   ```
6. **Watch the east-west traffic.** Segmentation controls lateral (east-west) movement, which most flat networks don't monitor at all; log and alert on cross-zone traffic that shouldn't happen.

### Cheatsheet

```
zone by trust + function
  DMZ (internet-facing) | user workstations | app/server tier | database tier
  management/admin (Tier 0) | OT/IoT | guest    -> sensitive systems get own zone

the rule: DEFAULT DENY between zones; allow only explicit, needed flows
  workstations -> app tier (specific ports)
  app tier     -> db tier (specific ports)
  db tier      -> initiates nothing outbound
  admin/Tier 0 -> reachable only from narrow defined sources
  OT/IoT, guest -> walled off from corporate

verify (the skipped step)
  from zone A, scan zone B -> is it actually denied, or just a VLAN with no ACL?
  nmap -sS -p- <other-zone>   ->  should be filtered/minimal
monitor east-west (lateral) traffic — flat nets don't, segmented ones must
```

### Reading the design/environment

- **A flat network (everything can reach everything)** = one compromise reaches it all; the single biggest lateral-movement enabler. Segmentation is the fix, starting with isolating the crown jewels.
- **VLANs with no enforcing ACLs/firewall between them** = the appearance of segmentation without the substance — hosts in different VLANs can still route to each other. Verify with a scan; this is the classic false sense of security.
- **Databases/DCs reachable from the user network** = a direct path from a phished workstation to the crown jewels; these belong in tightly-restricted zones.
- **OT/IoT on the corporate network** = fragile, unpatched devices sharing a broadcast domain with everything — a frequent pivot. Wall them off.
- **Verified default-deny between zones with monitored east-west traffic** = the good state; lateral movement now costs the attacker real effort and generates signals.

### The fix / best practice

- **Segment by trust and function with default-deny boundaries**, allowing only the specific flows each zone needs.
- **Isolate crown jewels and risky devices hardest** — databases, Tier 0, and OT/IoT get the tightest boundaries.
- **Verify enforcement**, don't trust the diagram — test reachability across boundaries and confirm a VLAN is backed by an ACL.
- **Monitor east-west traffic** so cross-zone attempts that shouldn't happen are visible.
- **Move toward least-privilege/zero-trust** over time (per-flow authorisation, microsegmentation) rather than trusting anything just because it's "inside".
- Re-verify after network changes — segmentation drifts as rules get added.

### Pitfalls

- **VLANs mistaken for segmentation.** A VLAN separates broadcast domains; without an enforcing ACL/firewall between VLANs, hosts still reach each other. Segmentation requires the enforced boundary, and you must test it.
- **Never verifying.** The diagram says isolated; the network says otherwise. Scan across boundaries to confirm.
- **Segmenting north-south, ignoring east-west.** Perimeter firewalls don't stop lateral movement inside; the internal boundaries are the point.
- **Leaving crown jewels reachable from user zones.** The whole value is keeping the sensitive systems away from the most-likely-compromised ones.
- **Set-and-forget.** Every new "allow this one flow" rule erodes segmentation; review periodically.

### References

- NIST SP 800-207 (Zero Trust Architecture) and segmentation guidance
- CIS Controls — network segmentation and boundary defence
- The AD tiered-admin-model skill (Tier 0 isolation) and red-team lateral-movement skill
- CWE-923 (improper restriction of communication channel)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.