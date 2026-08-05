---
name: edr-detection-logic
domain: 18-detection-engineering
description: Use when writing endpoint behavioural detections — the process, file, and behaviour patterns EDR telemetry exposes, and detecting attacker techniques rather than specific tools.
difficulty: intermediate
tags: [detection, edr, endpoint, behavioural, process]
tools: [sysmon, edr]
---

## Purpose

Endpoint Detection and Response telemetry — process creation, file and registry activity, network connections, module loads — is the richest source for catching attacker behaviour on hosts. But the value comes from detecting *behaviour* (the technique) rather than *tools* (a specific binary name), because tools change and behaviours are harder to alter. This skill covers writing endpoint detection logic that catches how attackers operate, using the deep telemetry EDR and Sysmon provide.

## When to use it

Building host-based detections, which are among the highest-fidelity you can write because endpoint telemetry sees what actually executes. It applies to commercial EDR and to Sysmon (free, on Windows), and pairs with the Sigma-writing skill (much of which targets endpoint telemetry).

## Procedure

1. **Detect behaviour, not tool names — the central principle.** A rule for `mimikatz.exe` is defeated by renaming the file; a rule for the *behaviour* (a process reading LSASS memory) catches the technique regardless of tool name. Aim your logic at the durable behaviour (Pyramid-of-Pain thinking applied to endpoint detection).
2. **Use process relationships.** The parent-child chain is one of the strongest signals — many attacks show as an anomalous lineage: Office spawning PowerShell/cmd, a web server spawning a shell, `services.exe` spawning something unexpected. Detect the suspicious *relationship*, not just the process.
3. **Detect command-line patterns.** Attacker commands have tells: encoded PowerShell (`-EncodedCommand`), download cradles, LOLBin abuse (legitimate binaries like `rundll32`, `regsvr32`, `mshta` used maliciously), and suspicious argument combinations. The command line is high-value endpoint telemetry.
4. **Detect key behaviours by their telemetry signature:**
   - **Credential access** — LSASS memory reads (the mimikatz behaviour, whatever the tool).
   - **Persistence** — Run-key/service/scheduled-task creation, autostart modifications.
   - **Injection** — remote-thread creation, suspicious memory allocation in another process.
   - **Discovery/execution** — sequences of recon commands, LOLBin execution chains.
5. **Tune for the environment.** Endpoint detections false-positive on legitimate admin and software behaviour; apply the false-positive discipline (understand the benign cause, exclude precisely) since endpoints are noisy.
6. **Ensure the telemetry exists.** These detections need the right endpoint logging — Sysmon with a good config, or EDR with the relevant events enabled. A rule for a telemetry source you don't collect never fires (the log-source-coverage skill).
7. **Map to ATT&CK and test** against real technique execution (atomic tests) so you know the behavioural rule actually catches it.

## Cheatsheet

```
CENTRAL: detect BEHAVIOUR (technique), not TOOL names
  bad:  Image = 'mimikatz.exe'   (renamed -> defeated)
  good: process reads LSASS memory  (the credential-access behaviour, any tool)

high-value endpoint signals
  PROCESS LINEAGE   Office->PowerShell ; webserver->shell ; services.exe->odd child
  COMMAND LINE      -EncodedCommand ; download cradles ; LOLBins (rundll32/regsvr32/mshta)
  CREDENTIAL ACCESS LSASS memory read
  PERSISTENCE       Run key / service / scheduled task / autostart creation
  INJECTION         remote thread / suspicious cross-process memory alloc
  DISCOVERY         recon command sequences

telemetry: needs Sysmon (good config) or EDR with the events enabled
  (rule for uncollected telemetry never fires — log-source-coverage)
tune: endpoints are noisy -> precise exclusions (reducing-false-positives)
map to ATT&CK ; test with atomic tests (real technique execution)
```

## Reading endpoint detections

- **A rule keyed on a tool name/hash** = brittle; renaming or recompiling defeats it, and it catches only that one tool. Rewrite toward the behaviour, which the attacker can't trivially change. This is the most common endpoint-detection weakness.
- **A process-lineage rule** (Office → PowerShell, webserver → shell) = high-fidelity; anomalous parent-child relationships are strong, durable signals that catch a class of attacks regardless of the specific payload.
- **A behavioural credential-access rule** (LSASS read) = catches the technique whatever tool is used — far more durable than a mimikatz-name rule.
- **A rule for telemetry you don't collect** = never fires; behavioural endpoint detection depends on the right logging (Sysmon config / EDR events) being on. Verify coverage.
- **An endpoint rule flooding on legit admin activity** = expected (endpoints are noisy); tune precisely rather than abandoning it, since the behaviour is worth detecting.
- **Behaviour-focused, telemetry-backed, tuned, ATT&CK-mapped, tested endpoint rules** = the high-fidelity detection endpoint data makes possible.

## Pitfalls

- **Detecting tools instead of behaviour.** Tool-name/hash rules are defeated by trivial changes (rename, recompile) and miss every other tool doing the same thing. Target the durable behaviour.
- **Ignoring telemetry requirements.** Behavioural rules need rich endpoint logging; without a good Sysmon config or the right EDR events, the rule has nothing to match. Confirm the source.
- **Untuned endpoint rules.** Endpoints generate huge volumes of legitimate activity; without precise tuning, behavioural rules flood the SOC. Apply false-positive discipline.
- **Not testing against real technique execution.** A behavioural rule is a hypothesis until it fires on an atomic test of the technique; validate it.
- **Over-narrow behavioural rules.** Matching one exact variant of a behaviour recreates the tool-name problem; aim for the technique's core signature, tolerant of variants.

## References

- Sysmon and a curated Sysmon config (e.g. SwiftOnSecurity/Olaf Hartong)
- MITRE ATT&CK and Atomic Red Team (for testing endpoint detections)
- The writing-sigma-rules, reducing-false-positives, log-source-coverage, and testing-detections skills
- The threat-intel pyramid-of-pain (behaviour > tools durability)
