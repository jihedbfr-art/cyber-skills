---
name: writing-sigma-rules
domain: 18-detection-engineering
description: Use when you want to write portable detection logic — a Sigma rule that expresses a detection once and converts to any SIEM's query language — and tune it to fire on the right thing.
difficulty: intermediate
tags: [detection, sigma, siem, rules, detection-as-code]
tools: [sigma, sigma-cli]
---

## Purpose

Sigma is to detections what a signature format is to antivirus: a vendor-neutral YAML way to describe "what does malicious look like in the logs", which then compiles to Splunk SPL, Elastic KQL, Sentinel, and the rest. Write the logic once, deploy it anywhere. This skill covers writing a Sigma rule that catches real activity without burying analysts in false positives.

## When to use it

When you have a detection idea — from a threat report, a hunt that found something, or an ATT&CK technique you lack coverage for — and want it as a maintainable, portable rule rather than a one-off SIEM query.

## Procedure

1. **Start from a specific behaviour, not a vague fear.** "Detect PsExec-style lateral movement" is testable; "detect hackers" isn't. Ideally tie it to an ATT&CK technique so coverage is trackable.
2. **Identify the log source and the fields** that expose the behaviour. You can only detect what you collect — confirm the telemetry exists before writing the rule (Windows Security 4688, Sysmon, EDR process events, etc.).
3. **Write the rule.** The core is `logsource` (where to look) and `detection` (what to match), with a `condition` tying selections together:
   ```yaml
   title: Suspicious Rundll32 Without Arguments
   status: experimental
   logsource:
     category: process_creation
     product: windows
   detection:
     selection:
       Image|endswith: '\rundll32.exe'
     filter:
       CommandLine|contains: '.dll'
     condition: selection and not filter
   falsepositives:
     - Legitimate software invoking rundll32 in unusual ways
   level: medium
   ```
4. **Build the condition to cut false positives.** The pattern `selection and not filter` is your main tool — match the suspicious shape, then subtract the known-good. Add `falsepositives` notes so whoever tunes it later knows what's expected.
5. **Convert and test** against real data before deploying. Compile to your SIEM and run it over historical logs to see what it would have fired on:
   ```
   sigma convert -t splunk rule.yml
   ```
6. **Validate it actually fires** on the real behaviour — run the technique in a lab (atomic test) and confirm the rule triggers. A rule that never fires on the thing it targets is worse than no rule, because it implies coverage you don't have.

## Cheatsheet

```yaml
# rule skeleton
title: <specific behaviour>
id: <uuid>
status: experimental        # -> test -> stable
logsource:
  category: process_creation   # or: file_event, network_connection, etc.
  product: windows
detection:
  selection:
    <field>|<modifier>: <value>     # endswith, startswith, contains, re
  filter:
    <field>: <known-good>
  condition: selection and not filter
falsepositives: [ ... ]
level: low|medium|high|critical
tags:
  - attack.t1218.011              # map to ATT&CK
```

```bash
# convert to a target SIEM
sigma convert -t splunk rule.yml
sigma convert -t elasticsearch rule.yml
# lint / validate
sigma check rule.yml
```

## Reading a rule's quality

- **Fires on the technique in a lab test** = real coverage. This is the pass/fail check — an untested rule is a hypothesis, not a detection.
- **A tight `condition` with sensible filters** = it'll survive contact with production logs. A bare `selection` with no filter usually floods the analyst.
- **Mapped to an ATT&CK technique** = you can track coverage and gaps across your detection set.
- **`falsepositives` documented** = maintainable; the next person can tune it without guessing what's expected.
- **`level` matched to real impact** = alerts get the attention they deserve; everything-is-critical trains analysts to ignore them.

## Deploying well (the "fix"/operationalisation)

- **Version it as code.** Sigma rules live in git, get reviewed, and deploy through CI — that's detection-as-code, the theme of this domain.
- **Promote through `status`**: experimental → test → stable as it proves itself against real data, so unvetted rules don't page anyone.
- **Tune iteratively.** Deploy in a low-noise mode first, watch the false positives, tighten the filter, then raise severity.
- **Track coverage** against ATT&CK so you're writing rules for gaps, not duplicating what you already catch (see mapping-to-attack).

## Pitfalls

- **Writing rules for telemetry you don't collect.** The rule compiles fine and never fires. Confirm the log source exists first.
- **No filter / too broad.** A `selection`-only rule on a common binary buries the SOC. Subtract known-good with `not filter`.
- **Never testing that it fires.** An untested rule is assumed coverage you don't actually have — validate with an atomic test.
- **Everything at `level: critical`.** Alert fatigue is a detection failure. Rate honestly.

## References

- Sigma specification and rule repository (SigmaHQ)
- sigma-cli / pySigma documentation
- MITRE ATT&CK (technique mapping)
- Atomic Red Team (for validating rules fire)
