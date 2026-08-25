---
format: "v2"
name: "soar-automation"
title: "Soar Automation"
title_fr: "Automatisation SOAR"
description: "Use when automating SOC work with SOAR — playbooks that handle the repetitive parts of triage and response so analysts focus on judgement, without automating away control of consequential actions."
description_fr: "À utiliser pour automatiser le travail du SOC avec le SOAR — des playbooks qui gèrent les parties répétitives du triage et de la réponse pour que les analystes se concentrent sur le jugement, sans automatiser le contrôle des actions à conséquences."
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

SOC analysts spend enormous time on repetitive, mechanical work — enriching alerts, gathering context, running the same lookups, closing known false positives. SOAR (Security Orchestration, Automation and Response) automates that mechanical work with playbooks, freeing analysts for the judgement that actually needs a human. This skill covers using SOAR to speed up and standardise SOC operations, and the crucial discipline of automating the right things — the repetitive and low-risk — while keeping humans in control of consequential decisions.

### When to use it

When the SOC is bogged down in repetitive manual work (visible in triage-time metrics and analyst load), or to standardise response so it's consistent regardless of who's on shift. It amplifies the alert-triage, enrichment, and use-case skills by automating their mechanical parts.

### Procedure

1. **Automate the repetitive, mechanical, high-volume work first.** The best SOAR candidates are tasks analysts do the same way every time: enrichment (asset/identity/threat-intel lookups — the enrichment skill), gathering context, deduplicating, and closing well-understood false positives. Automating these reclaims the most time with the least risk.
2. **Standardise response with playbooks.** Encode the response runbooks (from use-case development) as playbooks so every alert of a type is handled the same way, consistently, at machine speed — this is orchestration: tying together the tools (SIEM, EDR, ticketing, threat-intel) into one automated flow.
3. **Keep humans in the loop for consequential actions — the key discipline.** Automating enrichment and triage prep is safe; automatically *containing* (isolating a host, disabling an account, blocking) is powerful but risky — an automated action on a false positive can cause an outage or lock out a legitimate user. For consequential/irreversible actions, either require human approval in the playbook or reserve them for high-confidence conditions. Automate the *preparation*, gate the *action*.
4. **Design playbooks to augment, not replace, the analyst.** The goal is to hand the analyst a fully-enriched, context-rich alert with recommended actions — so they decide in seconds instead of gathering for minutes — not to remove them from the loop entirely. Judgement stays human.
5. **Test playbooks carefully** — a buggy automation runs its mistake at scale and speed. Test in a safe environment, and roll out with monitoring, especially for anything that takes action.
6. **Measure the impact.** Track time saved, consistency gained, and analyst load reduced (metrics skill). SOAR that doesn't measurably help — or that adds fragile complexity — isn't worth the maintenance.
7. **Maintain the playbooks.** Like detections, they rot as tools and processes change; own and review them.

### Cheatsheet

```
SOAR = automate the mechanical SOC work -> analysts do the JUDGEMENT

automate FIRST (repetitive, mechanical, high-volume, LOW-RISK)
  enrichment (asset/identity/intel lookups) | context gathering | dedup
  | closing well-understood false positives

standardise: encode response runbooks as playbooks (consistent, machine-speed)
  orchestration = tie tools together (SIEM+EDR+ticketing+intel) into one flow

KEY DISCIPLINE: humans-in-the-loop for CONSEQUENTIAL actions
  safe to auto:   enrichment, triage prep, context (reversible/no side effect)
  gate/approve:   containment (isolate host, disable account, block)
                  auto-action on a FALSE POSITIVE = outage / lockout
  -> automate the PREPARATION, gate the ACTION (or high-confidence only)

goal: augment the analyst (enriched alert + recommended actions), not replace
test carefully (buggy automation errs at SCALE) ; measure impact ; maintain
```

### Reading SOAR use

- **Analysts drowning in repetitive lookups and context-gathering** = the prime SOAR opportunity; automating enrichment and triage prep reclaims the most time at the lowest risk. Start here.
- **Automated containment on low-confidence detections** = risky; an automated isolate/disable/block on a false positive causes outages or locks out legitimate users. Gate consequential actions behind approval or high confidence.
- **Playbooks that remove the analyst entirely** = usually over-automation; judgement-requiring decisions handled by rigid automation cause bad outcomes on the cases that don't fit the pattern. Augment, don't replace.
- **A buggy playbook** = it makes its mistake at scale and speed; test thoroughly and monitor, especially action-taking playbooks.
- **SOAR that adds complexity without measurable benefit** = fragile overhead; measure time saved and load reduced, and drop automations that don't earn their maintenance.
- **Automated enrichment + standardised playbooks + gated consequential actions** = SOAR done right; faster, consistent triage with humans still in control.

### Pitfalls

- **Automating consequential actions without a gate.** Auto-containment on a false positive is an outage or a lockout; keep humans in the loop for isolate/disable/block, or restrict to high-confidence conditions. Automate preparation, gate action.
- **Over-automating judgement.** Removing the analyst from decisions that need judgement produces bad outcomes on edge cases; SOAR should augment, handing the analyst an enriched alert, not replace them.
- **Under-testing playbooks.** Automation errs at scale and speed; a bug affects many alerts fast. Test safely and monitor rollout.
- **Automating the wrong things.** Complex, judgement-heavy, or rare tasks are poor candidates; automate the repetitive, mechanical, high-volume, low-risk work.
- **Fire-and-forget playbooks.** They rot as tools and processes change; own, review, and measure them.

### References

- SOAR platform documentation (playbook design, orchestration)
- The SOC alert-triage-workflow, siem-use-case-development, and detection alert-enrichment skills
- The AI excessive-agency skill (same automate-preparation-gate-action discipline)
- SANS SOC automation resources

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.