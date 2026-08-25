---
format: "v2"
name: "evidence-preservation"
title: "Evidence Preservation"
title_fr: "Préservation des preuves"
description: "Use during an incident to capture volatile and persistent evidence in the right order, without contaminating it — so the investigation (and any legal case) holds up."
description_fr: "À utiliser pendant un incident pour capturer les preuves volatiles et persistantes dans le bon ordre, sans les contaminer, afin que l'investigation — et un éventuel dossier judiciaire — tienne la route."
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

An incident investigation is only as good as the evidence you preserved while responding. Move carelessly and you overwrite the memory that held the malware, or reboot away the network connections that showed the C2. This skill covers capturing evidence in the correct order, without contaminating it, so you can actually reconstruct what happened — and defend it later if the case goes legal.

It's the IR-side companion to the forensics domain: this is the fast, in-the-moment preservation; deep analysis comes later, on the copies you took here.

### When to use it

During active incident response, in parallel with containment — often the two are the same moment (isolate the host, but capture its volatile state first). Whenever findings might be challenged (legal, HR, insurance), preservation discipline is what makes them stand up.

### The order of volatility

Capture from most fleeting to most durable — the volatile stuff disappears on reboot or power-off, so it goes first:

1. **Memory (RAM)** and CPU/cache state — gone the instant power is lost; holds running processes, injected code, keys, network state.
2. **Network state** — active connections, ARP, routing, listening ports — changes second to second.
3. **Running processes and open files** — the live picture of what's executing.
4. **Disk** — persistent; survives reboot, so it can wait until after the volatile captures.
5. **Logs and archived data** — the most durable, and often already centralised.

### Procedure

1. **Decide what needs preserving** based on the incident and whether it may become a legal matter. If in doubt, preserve more — you can't go back for volatile data.
2. **Capture volatile evidence first, in order of volatility.** Take a **memory image** before powering off or isolating in a way that reboots. Then capture network state and process lists. Do this with minimal footprint — every action on the host changes it, so record what you run.
3. **Then image the disk** (bit-for-bit, hashed — see the forensics disk-imaging skill). Persistent data can wait behind the volatile captures.
4. **Preserve logs** from the host and from central systems (SIEM, cloud audit logs, firewall) — pull and protect them so retention or an attacker can't age them out.
5. **Hash everything you collect** and record the hashes — this is the integrity proof that the evidence wasn't altered after collection.
6. **Maintain chain of custody**: for each item, who collected it, when, from where, and every handoff since — an unbroken log. Store originals read-only; work on copies.
7. **Minimise contamination**: don't install tools onto the evidence system if avoidable, don't browse the filesystem casually (it changes access times), and record any action you do take on it.

### Cheatsheet

```
ORDER OF VOLATILITY (capture top-first, before power-off)
  1. RAM / memory image          <- most volatile, gone on reboot
  2. network state (conns, arp, ports)
  3. running processes / open files
  4. disk image (bit-for-bit, hashed)
  5. logs (host + central: SIEM, cloud, firewall)   <- most durable

for every item
  [ ] hash it (sha256) and record the hash
  [ ] chain of custody: who / when / where / handoffs
  [ ] store original read-only; analyse a copy
  [ ] log any action taken ON the evidence system

contamination rules
  don't reboot/power-off before memory capture
  don't install tools on the host if avoidable
  don't casually browse the FS (changes timestamps)
```

### Reading the situation

- **A host that will be investigated** must have its **memory captured before** any power-off or reboot — that's the single most common irrecoverable loss.
- **A fast-moving threat** forces a judgement: sometimes you must contain before a full memory capture. Record the decision and preserve what you can — a partial, documented capture beats none.
- **Anything that might go legal** raises the bar — strict chain of custody and hashing become mandatory, not optional. When unsure, treat it as if it will.
- **Logs with short retention** are evidence on a timer — pull and protect them early before they roll off or get tampered with.
- **A gap in the custody log** is what gets evidence thrown out later; an unexplained handoff can undo otherwise clean work.

### Pitfalls

- **Rebooting or powering off before capturing memory.** The most damaging preservation mistake — volatile evidence is gone for good.
- **Working on the original.** Analyse copies; keep originals read-only and hashed, or you contaminate the evidence you're trying to use.
- **No hashing.** Without integrity hashes you can't prove the evidence is unaltered — weak for investigation, fatal for legal.
- **Broken chain of custody.** One undocumented handoff can invalidate everything. Log every touch.
- **Contaminating the host.** Installing tools and browsing the filesystem changes it; minimise footprint and record what you do.

### References

- NIST SP 800-86 (Integrating Forensic Techniques into Incident Response)
- RFC 3227 (Guidelines for Evidence Collection and Archiving — order of volatility)
- SANS incident response and forensics resources
- ISO/IEC 27037 (handling digital evidence)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.