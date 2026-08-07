---
name: endpoint-hunting
domain: 20-threat-hunting
description: Use when hunting on endpoint telemetry — the process, persistence, and injection patterns that reveal compromise on hosts, using the richest data source available to a hunter.
difficulty: intermediate
tags: [threat-hunting, endpoint, edr, process, persistence]
tools: [edr, sysmon]
---

## Purpose

The endpoint is where attackers actually execute, persist, and act — which makes endpoint telemetry (process creation, file/registry activity, module loads, network connections) the richest source for hunting. Endpoint hunting looks for the process, persistence, and injection patterns that betray compromise on hosts. This skill covers hunting that telemetry effectively, complementing the network-focused hunts with the host view where the attacker's real activity happens.

## When to use it

Hunting for compromise on hosts, which is most hunting — the endpoint sees what actually ran, not just what crossed the network. It leans on EDR or Sysmon telemetry and pairs with the detection EDR-logic skill (same data, hunting rather than alerting).

## Procedure

1. **Hunt process execution and lineage.** The core endpoint data. Look for anomalous process trees (Office → PowerShell, a service spawning a shell, unexpected parent-child chains), rare processes (a binary seen on one host once), processes running from unusual locations (`%TEMP%`, `%APPDATA%`, `\Users\Public`), and suspicious command lines (the LOTL skill). Process lineage is one of the highest-fidelity endpoint signals.
2. **Hunt persistence mechanisms.** Attackers persist through predictable surfaces: registry Run keys, services, scheduled tasks, WMI subscriptions, startup folders, and (Linux) cron/systemd/authorized_keys. Hunt for new or unusual entries in these — a rarely-changing surface makes anomalies stand out (feeds IR eradication).
3. **Hunt injection and evasion.** Process injection, hollowing, and memory-only execution show in telemetry as remote-thread creation, suspicious cross-process memory operations, and processes with no disk-backed image (ties into memory forensics). These indicate advanced malware trying to hide.
4. **Hunt credential-access behaviour.** LSASS memory access, credential-store access, and dumping tools' behavioural signatures (the AD credential-dumping skill) — high-value because credential theft precedes lateral movement.
5. **Use rarity and stacking.** Across many endpoints, the *rare* thing stands out — a process, path, or command seen on one host out of thousands. Frequency analysis (the data-stacking skill) turns a fleet of endpoints into a way to surface the outlier.
6. **Baseline and combine signals.** Endpoints are noisy with legitimate activity; baseline normal (the baselining skill) and combine weak signals (rare process + unusual path + odd parent) into strong leads, as with LOTL hunting.
7. **Investigate and operationalise** — a confirmed endpoint compromise is an active intrusion (escalate to IR, dump the process for the malware domain); the pattern becomes an EDR detection.

## Cheatsheet

```
the endpoint = where attackers execute/persist/act -> RICHEST hunting data
  (EDR / Sysmon: process, file/registry, module loads, network)

hunt
  PROCESS + LINEAGE (highest fidelity): odd trees (Office->PS), rare process,
    unusual path (%TEMP%/%APPDATA%/Public), suspicious cmdline (LOTL)
  PERSISTENCE: Run keys, services, scheduled tasks, WMI subs, startup
    (Linux: cron/systemd/authorized_keys) — new/unusual entries stand out
  INJECTION/evasion: remote-thread, cross-process memory ops, no-disk-image process
  CREDENTIAL ACCESS: LSASS reads, dumping-tool behaviour (precedes lateral movement)

RARITY + STACKING: across the fleet, the rare thing (1 host of 1000s) = outlier
  frequency analysis -> surface the outlier (data-stacking)

BASELINE (endpoints noisy) + COMBINE weak signals (rare proc + odd path + odd parent = lead)
confirmed -> IR + dump for malware analysis ; pattern -> EDR detection
```

## Reading the hunt

- **An anomalous process tree** (Office spawning PowerShell, a service spawning cmd, an unexpected parent) = one of the highest-fidelity endpoint findings; the lineage betrays malicious execution regardless of the specific payload. A prime lead.
- **A rare process/path/command across the fleet** (seen on one host of thousands) = rarity is a powerful signal; the outlier that nothing else in the environment does is worth investigating. Stacking surfaces it.
- **A new persistence entry** (an unfamiliar Run key, service, scheduled task, or WMI subscription) = attacker persistence; these surfaces change rarely, so anomalies stand out, and it's exactly what IR must eradicate.
- **Injection signatures** (remote-thread creation, a process with no disk image) = advanced malware hiding in memory; endpoint telemetry catches what disk analysis misses.
- **LSASS access or credential-dumping behaviour** = credential theft, which precedes lateral movement; a high-value find that indicates the intrusion is advancing.
- **Endpoint noise from legitimate activity** = the challenge; baseline normal and combine weak signals, since a single event is often benign. Correlation and rarity separate signal from noise.

## Pitfalls

- **Not collecting rich endpoint telemetry.** Endpoint hunting depends on EDR/Sysmon process and command-line data; without it (or with default-config Sysmon), the richest hunting source is unavailable. Ensure the logging.
- **Hunting single events without context.** One endpoint event is often benign; the findings emerge from lineage, rarity, and combining weak signals. Correlate.
- **No baseline.** Endpoints generate huge legitimate volume; without a baseline of normal, anomalies drown. Baseline per host role.
- **Ignoring rarity/stacking.** Across a fleet, frequency analysis is one of the most powerful ways to surface the outlier; hunting host-by-host misses the fleet-wide rare thing.
- **Focusing only on process, missing persistence/injection/credential access.** These are where attackers hide and advance; hunt all the endpoint behaviours, not just execution.

## References

- The detection edr-detection-logic and log-source-coverage skills; Sysmon configs
- The data-stacking, anomaly-baselining, and living-off-the-land skills
- The forensics memory-forensics and AD dcsync skills
- MITRE ATT&CK (execution, persistence, defense evasion, credential access) and Atomic Red Team
