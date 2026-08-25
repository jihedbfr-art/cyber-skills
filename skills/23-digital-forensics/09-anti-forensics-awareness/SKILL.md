---
format: "v2"
name: "anti-forensics-awareness"
title: "Anti Forensics Awareness"
title_fr: "Sensibilisation à l'anti-forensic"
description: "Use when investigating a system where an attacker may have tried to destroy or falsify evidence — recognising log clearing, timestomping, wiping, and the traces these techniques leave."
description_fr: "À utiliser pour investiguer un système où un attaquant a pu tenter de détruire ou falsifier des preuves — reconnaître l'effacement de logs, le timestomping, le wiping, et les traces que laissent ces techniques."
domain: "23-digital-forensics"
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

Sophisticated attackers don't just break in — they try to erase or falsify the evidence of it. Anti-forensics is the set of techniques used to defeat investigation: clearing logs, altering timestamps, wiping files, hiding data, and disabling logging. This skill covers recognising these techniques and the traces they leave, because the crucial insight is that anti-forensics is rarely perfect — the *attempt* to destroy evidence usually leaves its own evidence, and absence where you expect data is itself a finding.

### When to use it

Any investigation where a capable adversary was involved. Applying it means reading the gaps and inconsistencies — not just what's present, but what's suspiciously *missing* or *altered*. It sharpens every other forensics skill by making you sceptical of clean-looking data.

### The techniques and their traces

- **Log clearing / deletion** — wiping event logs to hide activity. *Trace:* a cleared Windows Security log records event **1102** (audit log cleared); a suspiciously empty or truncated log, or a gap in otherwise-continuous logging, all point to clearing. The absence is the evidence.
- **Timestomping** — altering file timestamps to hide when something happened. *Trace:* on NTFS, `$STANDARD_INFORMATION` vs `$FILE_NAME` timestamp mismatches; times that are impossible (modified before created), suspiciously round (all zeros/identical), or inconsistent with other artefacts (prefetch, logs).
- **File wiping / secure deletion** — overwriting files to prevent recovery. *Trace:* wiping-tool artefacts (the tool's own execution shows in prefetch/amcache), patterns of overwritten space, and a conspicuous absence of files you'd expect.
- **Data hiding** — steganography, alternate data streams (ADS), hidden partitions, misnamed extensions. *Trace:* ADS enumeration, entropy analysis, file-type vs extension mismatches (the static-triage habit).
- **Disabling logging / auditing** — turning off the recording before acting. *Trace:* configuration changes to logging, a gap starting exactly when logging was disabled.
- **Living-off-the-land / minimal footprint** — using built-in tools and memory-only execution to avoid leaving disk artefacts. *Trace:* the value of memory forensics, since disk may be clean by design.

### Procedure

1. **Read absence as evidence.** The core mindset: a clean system where you expected activity is suspicious, not reassuring. A missing log, an empty history, a gap in a timeline — treat these as findings pointing at anti-forensics, and investigate *why* the expected evidence isn't there.
2. **Look for the anti-forensic act itself.** Destroying evidence is an action that often leaves its own trace — the log-clear event (1102), the wiping tool in prefetch, the timestamp-altering pattern. Hunt for these markers explicitly.
3. **Corroborate across independent sources.** Anti-forensics on disk is undone by evidence elsewhere — memory (which may still hold what was wiped from disk), network logs (external to the host, harder to clear), central SIEM (off-host, out of the attacker's reach), and cloud audit logs. Cross-source contradiction exposes the tampering.
4. **Cross-check timestamps** ($SI vs $FN, artefact-vs-artefact) to catch timestomping, and don't trust a single artefact's times.
5. **Assume off-host evidence survives.** The strongest counter to anti-forensics is evidence the attacker couldn't reach — logs forwarded to a SIEM, network captures, cloud trails. This is why centralised, off-host logging (the auditd/cloudtrail skills) is a forensic control, not just a detection one.
6. **Document what appears tampered** and the basis for that conclusion — it's often material to the investigation (destruction of evidence has legal weight) and shapes how much to trust the remaining data.

### Cheatsheet

```
core mindset: ABSENCE where you expect data = a finding, not reassurance.
              anti-forensics is rarely perfect -> the ATTEMPT leaves traces.

technique            trace to look for
-------------------  -----------------------------------------------------------
log clearing         Windows event 1102 ; empty/truncated log ; logging GAP
timestomping         $SI vs $FN mismatch ; impossible/round times ; artefact conflict
file wiping          wiping-tool in prefetch/amcache ; overwritten patterns ; missing files
data hiding          ADS ; steganography (entropy) ; extension vs real type mismatch
disabled logging     config change + gap starting exactly then
LOTL / fileless      minimal disk footprint -> lean on MEMORY forensics

counter: corroborate across INDEPENDENT sources the attacker couldn't reach
  memory (holds what was wiped from disk) | network logs | central SIEM | cloud audit
off-host logging = a forensic control (evidence beyond the attacker's reach)
```

### Reading the signs

- **A cleared or suspiciously empty log** (event 1102, a truncated auth.log) = deliberate log clearing; the clearing itself, and the gap it leaves, are evidence of an attempt to hide activity. Don't read an empty log as "nothing happened".
- **$SI/$FN timestamp mismatches or impossible times** = timestomping; the file's real timeline is being hidden, and the inconsistency both flags it and tells you not to trust those timestamps.
- **A wiping tool in execution artefacts** = evidence destruction attempted; even if the wiped files are gone, the *tool's* execution proves it happened.
- **A gap in a timeline starting exactly when logging changed** = logging was disabled to create a blind spot; the timing correlation is the tell.
- **Disk clean but memory/network tell a different story** = the attacker cleaned disk but couldn't reach memory or off-host logs; the contradiction exposes the whole operation. This cross-source corroboration is the strongest counter.
- **Everything suspiciously pristine on a system you have other reason to suspect** = possible thorough anti-forensics; escalate scepticism and lean on off-host and memory evidence.

### Pitfalls

- **Reading a clean system as innocent.** The biggest anti-forensics trap — absence of evidence on disk may mean thorough cleanup, not innocence. Where you expected activity and find none, that's a finding to chase.
- **Trusting timestamps.** Timestomping is common; always cross-check $SI vs $FN and against other artefacts before building a timeline on file times.
- **Relying only on the host.** Anti-forensics targets the host; memory, network, and off-host/SIEM/cloud logs are where the surviving evidence is. Corroborate off-host.
- **Missing the anti-forensic act.** Focusing on what's gone and forgetting that the destruction itself (1102, wiping tool, config change) left traces to find.
- **Not documenting suspected tampering.** Evidence destruction is material (and often legally significant); record what appears tampered and why.

### References

- SANS anti-forensics and DFIR resources
- MITRE ATT&CK — T1070 (Indicator Removal), T1070.006 (Timestomp), T1562 (Impair Defenses)
- The timeline-analysis, memory-forensics, windows-artefacts, and auditd-and-logging skills
- Windows event 1102 / Linux log-integrity references

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.