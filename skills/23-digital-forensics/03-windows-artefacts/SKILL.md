---
name: windows-artefacts
domain: 23-digital-forensics
description: Use when investigating a Windows system — the registry, event logs, prefetch, and other artefacts that reveal what ran, when, and who did it.
difficulty: intermediate
tags: [forensics, windows, registry, event-logs, artefacts]
tools: [eric-zimmerman-tools, autopsy]
---

## Purpose

Windows records an enormous amount about what happened on it — often without the user or attacker realising. Program execution, USB devices, file access, logons, and persistence all leave artefacts across the registry, event logs, and specialised files. This skill covers the Windows artefacts that answer the core investigative questions — what ran, when, and who did it — and where each one lives.

## When to use it

Investigating a compromised or suspect Windows host (the majority of enterprise endpoints), after acquiring the disk image (and ideally memory). It's the bread-and-butter of host forensics, and knowing which artefact answers which question is what makes an investigation efficient.

## The artefacts by question

**"What programs ran?"**
- **Prefetch** (`C:\Windows\Prefetch`) — records executed programs, run count, and last-run times.
- **Amcache / Shimcache** (registry) — evidence of program execution and presence, even for deleted binaries.
- **UserAssist** (registry) — GUI programs the user launched.

**"What happened and when?" (Event logs)**
- **Security log** — logons (4624/4625), privilege use, account changes; the core of who-did-what.
- **System / Application logs** — service installs, crashes, errors.
- **PowerShell / Sysmon logs** — script execution and detailed process/network telemetry if enabled.

**"Who did it / what did they touch?"**
- **Registry** — user activity (`NTUSER.DAT`), recently-opened files, typed paths, USB device history, network connections, persistence (Run keys).
- **LNK files & Jump Lists** — recently accessed files and their original paths.
- **$MFT / USN Journal** (filesystem) — file creation/modification/deletion timeline.

## Procedure

1. **Frame the investigative question** — what ran, when did X happen, who logged in, what was accessed, how did they persist. The question tells you which artefacts to pull; going artefact-by-artefact without a question wastes time.
2. **Parse the key artefacts with proven tools.** Eric Zimmerman's tools are the standard for parsing Windows artefacts (registry, prefetch, MFT, event logs, LNK) into readable output:
   ```
   # Eric Zimmerman tools: PECmd (prefetch), Registry Explorer/RECmd (registry),
   #   EvtxECmd (event logs), MFTECmd ($MFT), LECmd (LNK), JLECmd (jump lists)
   ```
3. **Establish execution** (prefetch + amcache + userassist), **timeline** (event logs + MFT), **user activity** (NTUSER registry, LNK/jump lists), and **persistence** (Run keys, services, scheduled tasks) — the four pillars most cases need.
4. **Correlate across artefacts.** No single artefact tells the whole story; a program in prefetch + a logon in the security log + a Run key in the registry together reconstruct "attacker logged in, ran X, persisted via Y". Correlation is where the investigation happens.
5. **Feed into a timeline** (the timeline-analysis skill) so events across artefacts line up chronologically.
6. **Handle as evidence** — parse copies from the image, maintain chain of custody, and document what each artefact showed.

## Cheatsheet

```
question -> artefact (know which answers which)

WHAT RAN
  Prefetch (C:\Windows\Prefetch)   executed programs, run count, last-run  [PECmd]
  Amcache/Shimcache (registry)     execution/presence, even deleted binaries
  UserAssist (registry)            GUI programs the user launched

WHEN / WHAT HAPPENED
  Security.evtx    logons 4624/4625, priv use, account changes   [EvtxECmd]
  System/App.evtx  service installs, errors
  PowerShell/Sysmon  script + process/network telemetry (if enabled)

WHO / WHAT TOUCHED
  Registry NTUSER.DAT   recent files, typed paths, USB history, Run keys  [RECmd]
  LNK / Jump Lists      recently accessed files + original paths   [LECmd/JLECmd]
  $MFT / USN Journal    file create/modify/delete timeline   [MFTECmd]

tools: Eric Zimmerman suite (the standard)
CORRELATE artefacts -> reconstruct the story ; feed a timeline
```

## Reading the artefacts

- **A malicious binary in prefetch/amcache** = proof it executed (and when/how often), even if the file was later deleted — execution evidence that survives cleanup. High value.
- **Logon events (4624/4625) around the incident** = who accessed the system and when, including failed attempts (brute force) and lateral movement (network logons from other hosts).
- **A Run key, service, or scheduled task pointing at a suspicious binary** = the persistence mechanism; exactly what IR needs to eradicate.
- **LNK/jump list entries** = files the user (or attacker) opened and their original locations, including from now-removed USB devices or network shares.
- **$MFT timestamps that don't line up** (a file created after it was "modified", timestamps that look manipulated) = possible timestomping (anti-forensics) — the inconsistency is the tell.
- **Correlated artefacts telling one story** = the reconstructed sequence (logon → execution → persistence → access); no single artefact does this alone.

## Pitfalls

- **Hunting artefacts without a question.** There are dozens; going through them aimlessly is slow. Let the investigative question (what ran / when / who / persistence) drive which you pull.
- **Relying on one artefact.** Prefetch shows execution but not who or why; correlate with logs and registry to get the full picture.
- **Assuming logs are complete.** PowerShell/Sysmon detail exists only if it was enabled; an attacker may also have cleared logs (a cleared security log — event 1102 — is itself a finding).
- **Trusting timestamps blindly.** Timestomping alters file times; cross-check `$STANDARD_INFORMATION` vs `$FILE_NAME` timestamps and other artefacts.
- **Parsing the live/original system.** Work from the forensic image; touching the original changes artefacts (access times, etc.).

## References

- Eric Zimmerman's tools (ericzimmerman.github.io) and SANS Windows Forensic Analysis (FOR500)
- SANS Windows forensics artefact posters
- The disk-imaging-and-hashing, memory-forensics, timeline-analysis, and chain-of-custody skills
- MITRE ATT&CK (persistence and execution artefacts)
