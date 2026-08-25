---
format: "v2"
name: "living-off-the-land"
title: "Living Off The Land"
title_fr: "Détournement d'outils légitimes (living off the land)"
description: "Use when hunting for attackers abusing legitimate built-in tools (LOLBins) — the PowerShell, WMI, and signed-binary abuse that blends in with normal admin activity."
description_fr: "À utiliser pour traquer les attaquants qui détournent des outils légitimes déjà présents sur le système (LOLBins) : abus de PowerShell, WMI et binaires signés qui se fondent dans l'activité d'administration normale."
domain: "20-threat-hunting"
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

Modern attackers increasingly avoid custom malware and instead abuse the tools already on the system — PowerShell, WMI, `certutil`, `rundll32`, `mshta`, and other signed, legitimate binaries (LOLBins). This "living off the land" (LOTL) evades antivirus (the tools are legitimate) and blends into normal administrative activity. Hunting for it means distinguishing malicious use of a legitimate tool from the many benign uses — a hard, high-value hunt because LOTL is how sophisticated actors stay hidden.

### When to use it

Hunting for sophisticated intrusions that leave little malware behind, and in environments where the noise of legitimate admin tooling hides attacker activity. It's endpoint-heavy and one of the harder hunts precisely because the signal (a legitimate tool) is identical to the noise on the surface — the malicious *use* is the difference.

### Procedure

1. **Understand the LOLBin landscape.** Know which legitimate binaries attackers abuse and how — the LOLBAS project catalogues them (download, execute, bypass). `certutil` downloading a file, `rundll32`/`regsvr32`/`mshta` executing remote code, `bitsadmin` transferring, PowerShell download cradles. Knowing the abuse patterns is what lets you spot them.
2. **Hunt the malicious *use*, not the tool.** The tool is legitimate, so you can't alert on its presence — you hunt anomalous *usage*: unusual command-line arguments, an unexpected parent process, an odd execution context. `certutil` is fine; `certutil -urlcache -f http://... file.exe` (downloading an executable) is not.
3. **Focus on command-line and process context — the key data.** LOTL detection lives in command-line arguments and process lineage (from the endpoint EDR skill). A LOLBin spawned by Office, or with download/execute arguments, or in a user context it shouldn't run in, is the signal. Rich command-line logging (Sysmon/EDR) is a prerequisite — you can't hunt what you don't log.
4. **Hunt PowerShell specifically.** It's the LOTL workhorse: encoded commands (`-EncodedCommand`), download cradles, `Invoke-Expression`, obfuscation, and suspicious module loads. PowerShell script-block logging is essential telemetry here.
5. **Baseline your environment's legitimate use.** Admins and software use these tools legitimately and heavily; the false-positive challenge is severe. Baseline normal LOLBin usage in your environment so anomalous use stands out — what's benign varies by org.
6. **Combine weak signals.** A single LOLBin execution is often benign; several weak signals together (an unusual LOLBin + odd parent + suspicious arguments + on an unexpected host) build a strong lead. LOTL hunting is about correlating faint signals.
7. **Investigate and operationalise** — confirmed LOTL abuse is a real intrusion (escalate to IR); the specific abuse pattern becomes a behavioural detection (the EDR-detection and operationalising skills).

### Cheatsheet

```
LOTL = abuse LEGITIMATE built-in tools -> evades AV (tools are signed) + blends w/ admin
  the tool is identical to noise; the malicious USE is the difference

know the LOLBins (LOLBAS project): certutil, rundll32, regsvr32, mshta, bitsadmin, wmic, PowerShell
  patterns: download (certutil -urlcache), execute remote (rundll32/mshta), transfer (bitsadmin)

hunt the USE, not the tool
  anomalous ARGUMENTS (certutil downloading an .exe) + unexpected PARENT (Office->cmd)
  + odd execution CONTEXT
KEY DATA: command-line args + process lineage (needs Sysmon/EDR cmdline logging — prereq)

PowerShell (the LOTL workhorse): -EncodedCommand, download cradles, IEX, obfuscation
  -> needs script-block logging

BASELINE legit use (admins/software use these heavily — severe FP challenge; varies by org)
COMBINE weak signals: odd LOLBin + odd parent + suspicious args + unexpected host = lead
confirmed -> IR ; pattern -> behavioural detection
```

### Reading the hunt

- **A LOLBin with malicious-pattern arguments** (`certutil` downloading an executable, `rundll32` executing remote code, an encoded PowerShell command) = the LOTL signal; the argument pattern is what distinguishes abuse from the legitimate tool. High-value when combined with context.
- **A LOLBin spawned by an unexpected parent** (Office → PowerShell, a web server → cmd) = strong; the process lineage betrays that the legitimate tool is being driven by something malicious.
- **Heavy legitimate LOLBin use** = the noise that makes this hard; admins and software use these constantly, so without a baseline the malicious use drowns. Baselining is essential and org-specific.
- **A single benign-looking LOLBin execution** = usually noise on its own; LOTL hunting works by *combining* weak signals (tool + parent + args + host context) into a strong lead.
- **Missing command-line/script-block logging** = you can't hunt LOTL at all — the malicious use lives in the arguments you're not collecting. This is a visibility prerequisite, not optional.
- **Correlated weak signals pointing at one host/user** = the reliable LOTL find; escalate and turn the pattern into a behavioural detection.

### Pitfalls

- **Alerting on the tool's presence.** These are legitimate binaries used constantly; you can't flag their existence. Hunt anomalous *use* — arguments, parent, context.
- **No command-line/script-block logging.** LOTL detection depends entirely on command-line and PowerShell telemetry; without it, the hunt is impossible. Ensure the logging first.
- **No baseline.** Legitimate LOLBin use is heavy and org-specific; without knowing your normal, malicious use is invisible in the noise. Baseline.
- **Single-signal hunting.** One LOLBin execution is usually benign; the malicious cases emerge from correlating multiple weak signals. Combine them.
- **Underestimating the difficulty.** LOTL is deliberately evasive and blends with admin work; expect a high false-positive challenge and lean on context and correlation.

### References

- LOLBAS project (lolbas-project.github.io) and the GTFOBins equivalent for Unix
- The detection edr-detection-logic, log-source-coverage, and malware analysing-scripts-and-macros skills
- MITRE ATT&CK — T1059 (Command and Scripting Interpreter), T1218 (System Binary Proxy Execution)
- The anomaly-baselining and endpoint-hunting skills

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.