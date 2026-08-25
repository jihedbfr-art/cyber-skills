---
format: "v2"
name: "alert-triage-workflow"
title: "Alert Triage Workflow"
title_fr: "Workflow de tri des alertes"
description: "Use when working a SOC alert queue — a repeatable path from raw alert to verdict (false positive, benign, or escalate) that stays consistent across analysts and shifts."
description_fr: "À utiliser pour traiter la file d'alertes d'un SOC — un cheminement reproductible de l'alerte brute au verdict (faux positif, bénin ou escalade) qui reste cohérent d'un analyste et d'un shift à l'autre."
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

A SOC lives or dies by how it handles the alert queue. Without a consistent method, verdicts depend on which analyst caught the alert and how tired they are. This skill is a repeatable triage workflow: the same steps, same questions, every alert — so the queue gets worked reliably and nothing real slips through because someone was busy.

### When to use it

Every alert, every shift. It's the backbone of SOC operations and the thing new analysts most need and most often lack. It feeds incident response (when triage says "escalate") and detection engineering (when triage keeps seeing the same false positive).

### Procedure

1. **Read what the alert actually says.** What rule fired, on what asset, for which user, at what time? Understand the detection's intent before reacting — a rule name isn't the whole story.
2. **Gather context around it.** Pull the surrounding activity: what else did that host/user do just before and after? A single alert is a data point; the context is the story. Enrich with asset criticality and user role.
3. **Ask the triage questions in order:**
   - Is this a **true positive** (the thing the rule targets really happened) or a **false positive** (benign activity that matched)?
   - If true, is it **malicious or benign-true** (the activity happened but was authorised — an admin, a sanctioned scan, a pentest)?
   - If malicious, **what's the scope and severity** so far?
4. **Reach a verdict:**
   - **False positive** → close with a reason, and if it recurs, flag the rule for tuning (feedback to detection engineering).
   - **Benign true positive** → close with the authorised explanation noted.
   - **Malicious / suspicious** → escalate into incident triage (that skill takes over on severity and response).
5. **Document as you go.** A short, factual note — what fired, what you checked, why you concluded what you did. The next analyst and any later investigation depend on it.
6. **Feed the loop.** Recurring false positives → rule tuning. Novel true positives → a new detection or a hunt. Triage isn't just clearing the queue, it's improving the sensors.

### Cheatsheet

```
triage path
  1. read     -> what rule, asset, user, time, intent?
  2. context  -> surrounding activity, asset value, user role
  3. classify -> false positive? benign-true? malicious?
  4. verdict  -> close (with reason)  |  escalate (to IR)
  5. document -> what you saw, checked, concluded
  6. feedback -> recurring FP -> tune rule; novel TP -> new detection/hunt

the three outcomes
  false positive   benign activity matched the rule      -> close + maybe tune
  benign true      real activity, but authorised          -> close + note why
  malicious/susp.  real and unauthorised                  -> escalate to IR

close-out note must answer: what fired, what you checked, why this verdict
```

### Reading an alert

- **Context that explains it away** (a change ticket, a known admin task, a scheduled job) usually makes it a benign true positive — but confirm the explanation, don't assume it.
- **The same alert firing across many hosts/users** shifts it from "one alert" toward "campaign or broken rule" — investigate the pattern, don't close each one in isolation.
- **An alert on a high-value asset or privileged account** deserves more scrutiny before you call it benign — the cost of a wrong "false positive" here is highest.
- **"I can't tell yet"** means gather more context or escalate; it doesn't mean close-and-hope. An uncertain alert on a critical asset goes up, not away.
- **A recurring false positive** is a detection defect, not just an annoyance — closing it silently for the hundredth time is how real alerts get lost in the noise.

### Making it stick (the operational fix)

- **Runbooks per alert type** so triage steps are consistent regardless of who's on shift — codify the context to gather and the questions to ask.
- **Automate enrichment** (asset, identity, threat-intel lookups) so analysts start with context instead of gathering it by hand — the SOAR skill covers this.
- **Close the feedback loop formally**: track false-positive rates per rule and route the worst offenders to detection engineering for tuning.
- **Measure honestly** (time-to-triage, escalation accuracy) to spot where the queue or the analysts are overloaded.

### Pitfalls

- **Closing false positives without tuning.** The rule keeps firing, the noise keeps growing, and eventually a real alert drowns in it. Feed recurring FPs back.
- **Triage by rule name alone.** The label is a hint, not a verdict. Context decides.
- **Inconsistent verdicts.** Without a workflow, the same alert gets closed by one analyst and escalated by another. Runbooks fix this.
- **No documentation.** An undocumented close is invisible to the next investigation — and to the postmortem when it turns out to have mattered.

### References

- NIST SP 800-61r2 (incident handling, triage phase)
- SANS SOC and analyst workflow guidance
- MITRE ATT&CK (for classifying what an alert maps to)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.