---
format: "v2"
name: "hypothesis-driven-hunting"
title: "Hypothesis Driven Hunting"
title_fr: "Chasse aux menaces guidée par hypothèses"
description: "Use when you want to proactively hunt for threats the alerts missed — framing a testable hypothesis, searching the telemetry to prove or kill it, and turning findings into detections."
description_fr: "À utiliser pour traquer proactivement les menaces passées inaperçues par les alertes : formuler une hypothèse testable, interroger la télémétrie pour la confirmer ou l'infirmer, puis transformer les découvertes en détections."
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

Threat hunting is the assumption that something got past the alerts, and a structured search to find it. The discipline that separates hunting from aimless log-browsing is the **hypothesis**: a specific, testable statement of what an attacker would have done and what trace it would leave. This skill covers framing that hypothesis and running the hunt to a real conclusion.

### When to use it

When you want proactive coverage beyond what your detections catch — after a relevant threat report, when you suspect a gap, or on a regular cadence for a mature SOC. It complements detection engineering: hunts find the unknown, and repeatable findings become new detections.

### Procedure

1. **Form a hypothesis that's specific and testable.** Not "are we breached?" but "if an attacker established persistence via a scheduled task, I'd see task creation events with suspicious command lines on these hosts." Ground it in an ATT&CK technique, a threat report, or your own environment's risk.
2. **Decide what would prove or disprove it.** Name the data source and the signal in advance: which log, which fields, what pattern means "found it" versus "clean". Committing to this before searching stops you from rationalising whatever you happen to see.
3. **Confirm you have the telemetry.** If the logs that would show the behaviour aren't collected, the hunt can't conclude — that gap is itself a finding (a visibility gap to fix). Don't run a hunt you can't answer.
4. **Search the data** for the pattern. Start broad to establish what normal looks like, then narrow to the anomalies. Baselining is half the work — you can't spot abnormal without knowing normal.
5. **Investigate the hits.** Each candidate is triaged like an alert: benign, authorised, or malicious. Chase the interesting ones across hosts and time to build the story.
6. **Conclude explicitly.** Every hunt ends one of three ways: found a threat (→ incident response), found nothing (hypothesis not supported, coverage confirmed), or found a **gap** (missing telemetry or a detection that should have fired). All three are valuable — a "clean" hunt with good data is real assurance.
7. **Operationalise.** If the hunt found something repeatable, write a detection for it (Sigma skill) so you never have to hunt for that exact thing again. The point of hunting is to shrink the space of the unknown.

### Cheatsheet

```
the hunt loop
  1. hypothesis ... specific + testable, tied to ATT&CK / a report
  2. prove/kill ... which data source + signal decides it (chosen up front)
  3. telemetry .... do we even collect it? (no data = visibility gap finding)
  4. baseline ..... establish normal, then hunt the anomalies
  5. investigate .. triage hits: benign / authorised / malicious
  6. conclude ..... threat found | clean (assurance) | gap found
  7. operationalise found something repeatable? -> write a detection

a good hypothesis
  BAD:  "look for suspicious activity"
  GOOD: "an attacker using WMI for lateral movement would leave
         wmiprvse.exe spawning cmd/powershell on the target host"
```

### Reading the hunt

- **A hit that survives triage** (unexplained, unauthorised, matches the technique) = escalate to incident response immediately; the hunt just became an investigation.
- **Clean results with solid telemetry** = genuine assurance for that hypothesis. Record it — "we hunted for X, had the data, found nothing" is a real risk statement.
- **Can't conclude because the data isn't there** = a visibility gap. That's often the most valuable output of an early hunt: you learn what you can't see.
- **A pattern you found by hand that recurs** = a detection waiting to be written. If you hunted it once, automate it.
- **Beware confirmation bias**: deciding the signal after you've looked lets you explain away anything. Fix the prove/kill criteria before searching.

### Making it repeatable (the "fix")

- **Log the hypothesis, method, and outcome** for every hunt, so coverage is trackable and hunts aren't repeated blindly.
- **Turn findings into detections** — the maturity signal of a hunting programme is that it feeds detection engineering, steadily converting "hunt for it" into "alert on it".
- **Track coverage against ATT&CK** so hunts target gaps rather than well-covered techniques.
- **Fix visibility gaps** the hunts expose; better telemetry makes every future hunt (and detection) stronger.

### Pitfalls

- **Vague hypotheses.** "Find bad stuff" isn't huntable. Without a testable statement and a data source, you're browsing logs, not hunting.
- **Deciding the criteria after looking.** Confirmation bias turns any result into "success". Commit to prove/kill up front.
- **Hunting without the telemetry.** You can't conclude; recognise the gap and fix it instead of forcing an answer.
- **Finding something and not detecting it.** A hunt that finds a technique but produces no lasting detection means you'll hunt the same thing forever. Operationalise it.

### References

- MITRE ATT&CK (technique-driven hypotheses)
- The PEAK / TaHiTI threat hunting frameworks
- SANS threat hunting resources
- Atomic Red Team (to generate the activity you're learning to hunt)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.