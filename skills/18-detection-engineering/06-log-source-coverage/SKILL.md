---
format: "v2"
name: "log-source-coverage"
title: "Log Source Coverage"
title_fr: "Couverture des sources de logs"
description: "Use when assessing whether you collect the telemetry your detections need — mapping what you can and can't see, because you can only detect what you're logging."
description_fr: "À utiliser pour évaluer si vous collectez la télémétrie dont vos détections ont besoin — cartographier ce que vous pouvez voir et ce qui vous échappe, car on ne détecte que ce que l'on journalise."
domain: "18-detection-engineering"
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

Every detection rests on a log source — and a rule for telemetry you don't collect never fires, giving you the illusion of coverage with none of the substance. Log-source coverage is knowing what you can and can't see, so your detection strategy is grounded in reality. This skill covers assessing telemetry coverage and closing the visibility gaps that silently undermine detection. It's the foundation the other detection skills stand on: no telemetry, no detection.

### When to use it

Before writing detections (confirm the source exists), when assessing detection coverage (a gap may be a *visibility* gap, not a rule gap), and when planning what telemetry to collect. It sits underneath ATT&CK coverage mapping — a technique you "can't detect" is often one you can't *see*.

### Procedure

1. **Inventory your log sources and what each provides.** What telemetry are you actually collecting — endpoint (Sysmon/EDR), network (firewall, DNS, proxy, IDS), authentication/identity, cloud audit logs, application logs? For each, know what events it captures. This inventory is the map of what's detectable.
2. **Map telemetry to detection needs — via ATT&CK.** ATT&CK data sources link techniques to the telemetry needed to detect them. Cross-reference the techniques you care about against the sources you have: which techniques could you detect *if you wrote the rule*, and which you can't see at all?
3. **Distinguish visibility gaps from rule gaps — the key insight.** An uncovered technique on the ATT&CK map has two possible causes: you have the telemetry but haven't written the rule (a rule gap, fixable by detection engineering), or you don't collect the telemetry at all (a visibility gap, fixable only by adding logging). These need completely different fixes; confusing them wastes effort.
4. **Assess telemetry quality, not just presence.** Having a log source isn't enough — is it complete (all hosts, not a sample?), is it configured for depth (Sysmon with a good config vs bare defaults?), and is it retained long enough to hunt/investigate? Shallow or partial telemetry is a partial gap.
5. **Prioritise closing gaps by value.** Some telemetry unlocks many detections (endpoint process/command-line data, authentication logs, DNS) — high-value coverage. Prioritise the sources that enable the most relevant detections for your threat model.
6. **Feed the gaps back.** Visibility gaps become logging/tooling requests (deploy Sysmon, enable cloud audit logs, forward a source to the SIEM); rule gaps become detection-engineering work. Route each correctly.

### Cheatsheet

```
core truth: a rule for telemetry you DON'T collect never fires.
            you can only detect what you're logging.

1. INVENTORY sources + what each captures
   endpoint (Sysmon/EDR) | network (fw/DNS/proxy/IDS) | auth/identity
   | cloud audit | application logs

2. MAP to detection needs via ATT&CK DATA SOURCES
   which techniques could you detect if you wrote the rule? which can't you SEE?

3. VISIBILITY gap vs RULE gap (the key distinction — different fixes!)
   have telemetry, no rule    -> RULE gap    -> write detection
   don't collect telemetry    -> VISIBILITY gap -> add logging (only fix)
   confusing them = wasted effort

4. QUALITY not just presence
   complete (all hosts)? | configured for depth (good Sysmon config)? | retained long enough?

5. prioritise gaps by value: endpoint cmdline / auth / DNS unlock MANY detections
6. route: visibility gap -> logging request ; rule gap -> detection engineering
```

### Reading coverage

- **An ATT&CK technique you "can't detect"** = check *why* first: is it a rule gap (you have the data, just no rule) or a visibility gap (you don't collect the data)? This distinction determines the fix, and getting it wrong means writing a rule that can never fire.
- **A high-value telemetry source missing** (no endpoint process/command-line logging, no DNS logs, no cloud audit) = a large blind spot; many detections are impossible without it. These are the priority visibility gaps to close.
- **A log source present but shallow** (Sysmon on default config, logs on only some hosts, short retention) = partial coverage that overstates what you can see; a rule may fire on some hosts and miss others. Assess quality, not just presence.
- **A rule written for a source you don't fully collect** = it works where the telemetry exists and silently misses everywhere it doesn't — a dangerous false sense of coverage.
- **Visibility gaps routed as logging requests and rule gaps as detection work** = the two problems handled correctly; conflating them is the common waste.
- **An honest telemetry map showing what's detectable** = the grounding the whole detection programme needs.

### Pitfalls

- **Writing rules for telemetry you don't collect.** The rule validates and deploys but never fires — coverage in name only. Confirm the source exists and is collected before writing.
- **Confusing visibility gaps with rule gaps.** They have completely different fixes (add logging vs write a rule); mistaking one for the other wastes effort and leaves the real gap open.
- **Assuming presence equals coverage.** A source on some hosts, on shallow config, or with short retention gives partial visibility that overstates coverage. Assess completeness, depth, and retention.
- **Ignoring the foundation.** Detection engineers who focus only on rules while blind spots go unaddressed build on sand; telemetry is the substrate.
- **Not prioritising by value.** Closing a low-value visibility gap while high-value ones (endpoint, auth, DNS) stay open misallocates effort.

### References

- MITRE ATT&CK Data Sources (technique-to-telemetry mapping)
- Sysmon configs and EDR telemetry documentation
- The mapping-to-attack, edr-detection-logic, and writing-sigma-rules skills
- The SOC log-pipeline-design skill (getting sources in)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.