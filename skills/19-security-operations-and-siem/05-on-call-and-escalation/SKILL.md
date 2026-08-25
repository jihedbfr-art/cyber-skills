---
format: "v2"
name: "on-call-and-escalation"
title: "On Call And Escalation"
title_fr: "Astreinte et escalade"
description: "Use when setting up SOC on-call and escalation — the runbooks, tiers, and handoffs that make sure serious alerts reach the right person fast and nothing falls through at 3am."
description_fr: "À utiliser pour mettre en place l'astreinte et l'escalade du SOC — les runbooks, niveaux et transmissions qui garantissent qu'une alerte grave atteint la bonne personne rapidement, sans rien laisser passer à 3h du matin."
domain: "19-security-operations-and-siem"
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

A detection is only useful if the right person acts on it in time — and at 3am, tired, mid-flood of alerts, that depends entirely on process. On-call and escalation define who handles what, when to wake someone up, and how a serious alert climbs to the people who can act. This skill covers building that structure so a critical alert doesn't sit in a queue and a minor one doesn't wake an executive. It's the operational plumbing that turns detections into timely response.

### When to use it

Setting up SOC operations, or fixing a SOC where serious alerts get missed or minor ones over-escalate. It connects the triage workflow (deciding severity) to actual human response, and feeds incident response when an alert becomes an incident.

### Procedure

1. **Define escalation tiers and criteria.** Typically a tiered model: Tier 1 analysts triage and handle routine alerts; Tier 2 takes deeper investigation; and serious findings escalate to incident response / management. The key is clear, objective criteria for *when* something escalates from one tier to the next — vague criteria mean either everything escalates (fatigue) or nothing does (missed incidents).
2. **Write runbooks per alert type.** For each significant alert, a runbook says how to triage it and, critically, when to escalate and to whom. This makes response consistent regardless of which analyst (or how experienced) catches it, and is what lets someone act correctly at 3am without improvising (ties into use-case runbooks and IR playbooks).
3. **Set up on-call coverage.** Define who's reachable outside business hours, the rotation, and the paging mechanism. Cover the gaps — an alert at 3am with no one on call is a detection that does nothing until morning.
4. **Match escalation urgency to severity.** A critical alert (active compromise) pages someone immediately; a medium one waits for the queue; a low one is logged. Getting this calibration right is what prevents both alert fatigue (everything pages) and missed incidents (nothing pages).
5. **Define clear handoff points to incident response.** When an alert becomes an incident, the transition to IR must be defined — who declares it, who takes over, what information passes. A fumbled SOC-to-IR handoff loses time exactly when it's most costly (ties into the IR triage skill).
6. **Guard against alert fatigue in the escalation design.** If too much escalates, on-call gets burned out and starts ignoring pages — the escalation equivalent of a noisy detection. Tune the criteria (and the detections feeding them) so escalations are meaningful.
7. **Test and review.** Run through the escalation for a serious scenario (a tabletop) to confirm it works — the right person is reached, the runbook is followable. Review after real incidents where escalation was slow or wrong.

### Cheatsheet

```
a detection is useless if the right person doesn't act in time -> process decides

TIERS + criteria
  T1 triage/routine -> T2 deeper investigation -> IR/management (serious)
  CLEAR objective escalation criteria (vague = everything escalates OR nothing does)

RUNBOOKS per alert type: how to triage + WHEN to escalate + TO WHOM
  -> consistent response, any analyst, at 3am, no improvising

ON-CALL: who's reachable off-hours, rotation, paging ; cover the gaps
  (3am alert, no one on call = detection does nothing till morning)

URGENCY matches SEVERITY
  critical (active compromise) -> page NOW | medium -> queue | low -> log
  wrong calibration = alert fatigue (all page) OR missed incidents (none page)

HANDOFF to IR defined: who declares, who takes over, what info passes
  (fumbled SOC->IR handoff loses time when it's most costly)

guard fatigue: too much escalating -> on-call burnout -> ignored pages
test (tabletop) + review after slow/wrong escalations
```

### Reading the setup

- **Vague or missing escalation criteria** = either everything escalates (on-call burnout, ignored pages) or serious alerts sit in a queue (missed incidents). Clear objective criteria are what make escalation work; this is the most common failure.
- **A gap in on-call coverage** = alerts during that window do nothing until someone's back; a 3am critical with no pager is an undetected-in-practice incident. Cover the hours.
- **Severity/urgency mismatch** (criticals queuing, lows paging) = the calibration is off, causing either fatigue or delay. Match paging to real severity.
- **No runbooks** = response depends on who's on shift and how experienced; the same alert gets handled well by one analyst and mishandled by another. Runbooks standardise it.
- **A fumbled SOC-to-IR handoff** = lost time at the worst moment; define who declares and how the transition works.
- **Over-escalation burning out on-call** = the escalation version of alert fatigue; pages get ignored, and the real one is missed. Tune criteria and the feeding detections.
- **Clear tiers, runbooks, covered on-call, calibrated urgency, defined IR handoff** = serious alerts reach the right person fast and nothing falls through.

### Pitfalls

- **Vague escalation criteria.** The core failure — without objective "escalate when X" rules, escalation is inconsistent, causing both fatigue and misses. Define clear criteria.
- **On-call gaps.** Uncovered hours mean alerts wait; a detection nobody's paged for is effectively off. Ensure round-the-clock coverage where the risk warrants.
- **Miscalibrated urgency.** Paging on lows burns out on-call; queuing criticals delays response. Match urgency to severity.
- **No runbooks.** Response quality then depends on individual analysts; at 3am, improvising is error-prone. Write per-alert runbooks.
- **Undefined IR handoff.** The SOC-to-IR transition is where time gets lost if it's not planned; define it.
- **Over-escalation.** Too many escalations burn out responders until they tune out — then the real one is missed. Keep escalations meaningful.

### References

- The SOC alert-triage-workflow, siem-use-case-development, and shift-handover skills
- The incident-response domain (triage and playbook handoff)
- SANS SOC operations and on-call resources
- SRE on-call practices (adapted for security operations)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.