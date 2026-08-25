---
format: "v2"
name: "shift-handover"
title: "Shift Handover"
title_fr: "Passation d'équipe"
description: "Use when handing over between SOC shifts — the structured handoff that ensures ongoing investigations, watch items, and context transfer cleanly so nothing falls through the cracks."
description_fr: "À utiliser pour la passation entre équipes du SOC — un transfert structuré qui garantit que les investigations en cours, les points de surveillance et le contexte passent proprement, sans rien perdre en route."
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

A 24/7 SOC runs on shifts, and every shift change is a moment where things fall through the cracks — an ongoing investigation forgotten, a watch item nobody follows up, context lost between the analyst who saw the early signs and the one who'd see the next. Shift handover is the structured transfer that keeps operations continuous across those boundaries. It's unglamorous, but a fumbled handover is how a slow-burn incident gets missed — the early indicators seen on one shift, the payoff on another, and no one connecting them.

### When to use it

Every shift change in a 24/7 or multi-shift SOC. It's a routine process, but doing it well is what maintains continuity of awareness — and doing it badly is a recurring, quiet source of missed incidents.

### Procedure

1. **Hand over ongoing investigations with full context.** Any alert or incident still open must transfer with its state: what's been checked, what's suspected, what's pending, and what the next step is. The incoming analyst should be able to pick it up without re-deriving everything — lost investigation context is the most damaging handover gap.
2. **Communicate watch items.** Things that aren't yet incidents but bear watching — a host behaving slightly oddly, a user flagged for follow-up, a maintenance window that might generate benign alerts, a known-ongoing situation. These are exactly what gets forgotten across shifts and later turns out to matter.
3. **Pass on environmental context.** Planned changes, ongoing maintenance, known false-positive sources active right now, and any temporary conditions (a pentest in progress, a deployment) so the incoming shift interprets alerts correctly rather than chasing expected noise or dismissing real signals.
4. **Use a structured handover format**, not just a verbal chat. A consistent handover document/log (open items, watch items, environmental notes, escalations pending) ensures nothing is skipped and creates a record. Verbal-only handovers lose things.
5. **Make it a two-way handoff with acknowledgement.** The outgoing analyst briefs, the incoming analyst confirms understanding and asks questions. A one-way dump that the incoming shift half-reads isn't a handover.
6. **Log it for continuity and pattern-spotting.** A written handover log lets a later shift (or an investigation) see the thread of a slow-developing situation across days — the early signs on Monday connecting to the payoff on Wednesday. This cross-shift continuity is how slow-burn incidents get caught.
7. **Keep it focused.** Handover should transfer what matters (open/watch/context), not recite every closed alert; an overlong handover buries the important items.

### Cheatsheet

```
every shift change = a moment things fall through the cracks
  slow-burn incident: early signs one shift, payoff another, no one connects them

hand over
  OPEN INVESTIGATIONS   state: checked / suspected / pending / NEXT STEP
                        (incoming picks up without re-deriving — biggest gap if lost)
  WATCH ITEMS           not-yet-incidents to follow up (odd host, flagged user,
                        situation in progress) — exactly what gets forgotten
  ENVIRONMENTAL         planned changes, maintenance, active FP sources, pentest/deploy
                        (interpret alerts right, don't chase expected noise)

format: STRUCTURED handover doc/log (open/watch/context/escalations), not verbal-only
two-way: brief + ACKNOWLEDGE + questions (not a one-way dump)
LOG it: cross-shift continuity -> slow-developing situations visible across days
focused: transfer what matters, not every closed alert
```

### Reading the handover

- **An ongoing investigation handed over with clear state and next step** = clean continuity; the incoming analyst continues seamlessly. Lost investigation context (what was checked, what's next) is the most damaging gap — it forces re-work or, worse, a dropped thread.
- **Watch items communicated** = the not-yet-incidents get followed up; these are precisely what falls through cracks across shifts and later proves important. Forgetting them is how slow-burn situations get missed.
- **Missing environmental context** (an active pentest, a deployment, a known noisy source right now) = the incoming shift misreads alerts — chasing expected noise or, dangerously, dismissing a real signal as "probably the maintenance". Pass it on.
- **Verbal-only handover** = things get lost; without a structured record, items are skipped and there's no continuity thread. Use a handover document.
- **A one-way dump** = the incoming analyst half-absorbs it; without acknowledgement and questions, understanding isn't confirmed. Make it two-way.
- **A handover log spanning days** = lets a later shift connect a slow-developing thread; this cross-shift record is how multi-day incidents get caught.
- **Structured, acknowledged, logged handover of open/watch/context** = continuity maintained; nothing falls through.

### Pitfalls

- **Losing investigation context.** Handing over an open case without its state and next step forces re-work or drops the thread — the most damaging handover failure. Transfer full context.
- **Forgetting watch items.** Not-yet-incidents that bear watching are exactly what gets lost across shifts and later matters. Communicate them explicitly.
- **Missing environmental context.** Without knowing what's normal-right-now (maintenance, pentest, active noise), the incoming shift misinterprets alerts. Pass on the temporary conditions.
- **Verbal-only handovers.** They lose items and leave no record; use a structured format and log it.
- **One-way dumps.** A brief the incoming analyst doesn't confirm isn't a real handover; make it two-way with acknowledgement.
- **Overlong handovers.** Reciting every closed alert buries the important open/watch items; keep it focused on what matters.

### References

- The SOC alert-triage-workflow and on-call-and-escalation skills
- The incident-response communication-during-incidents skill (handoff discipline)
- SANS SOC operations and shift-management resources
- Aviation/healthcare handover practices (adapted — same continuity discipline)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.