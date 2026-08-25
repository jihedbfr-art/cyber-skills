---
format: "v2"
name: "blameless-postmortem"
title: "Blameless Postmortem"
title_fr: "Post-mortem sans recherche de coupable"
description: "Use after an incident is resolved to write a postmortem that finds the systemic causes and drives real fixes — without blaming individuals, so people stay honest."
description_fr: "À utiliser une fois l'incident résolu, pour rédiger un post-mortem qui identifie les causes systémiques et débouche sur de vraies corrections — sans blâmer d'individus, afin que chacun reste honnête."
domain: "22-incident-response"
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

The postmortem is where an incident pays for itself — if it produces honest analysis and changes that prevent recurrence. The word "blameless" isn't softness; it's what makes the analysis accurate. When people fear being blamed, they hide the details you most need, and you fix the wrong thing. This skill covers running a postmortem that gets the truth and turns it into durable improvements.

### When to use it

After an incident is resolved (the last phase of the IR lifecycle), for anything beyond the trivial. It closes the loop: containment/eradication fixed *this* incident; the postmortem stops the *next* one.

### Why blameless

Systems fail because of systemic weaknesses — missing guardrails, unclear procedures, alerts nobody saw — not because one person was careless. If someone could make a mistake that caused an incident, the real problem is that the system *allowed* the mistake to have that impact. Blaming the individual fixes nothing and teaches everyone to withhold information next time. Blameless analysis assumes people acted reasonably with what they knew, and asks why the system let a reasonable action lead here.

### Procedure

1. **Build an accurate timeline.** What happened, when, in order — detection, response actions, key decisions, resolution. Draw on the incident log and communication log. Facts first, judgement later.
2. **Establish impact.** What was affected, for how long, and the real cost (downtime, data, users, effort). This sizes the incident honestly and justifies the fixes.
3. **Find the contributing causes, systemically.** Ask *why* repeatedly, but aim each "why" at the system, not the person. "The alert was missed" → *why?* "It was one of 200 that day" → the real cause is alert fatigue, not the analyst. Most incidents have several contributing factors, not one root cause.
4. **Capture what went well**, not just what failed — the detection that worked, the runbook that helped. You want to keep and reinforce those, and it keeps the review balanced.
5. **Produce concrete, owned action items.** Each fix has an owner and a due date, and addresses a systemic cause (a guardrail, a detection, a process change), not "be more careful". Vague or ownerless actions never happen.
6. **Keep the language blameless** throughout the document — describe roles and systems ("the on-call engineer", "the deploy process"), not names-as-culprits. This is what keeps future postmortems honest.
7. **Share and follow up.** Circulate the postmortem so others learn, and *track the action items to completion* — an unimplemented postmortem is the same incident waiting to recur.

### Cheatsheet

```
postmortem structure
  1. summary        what happened, in a paragraph
  2. timeline       detection -> actions -> decisions -> resolution (with times)
  3. impact         scope, duration, cost (downtime/data/users)
  4. contributing causes   systemic, via "why?" aimed at the system
  5. what went well  keep and reinforce these
  6. action items   each: owner + due date + addresses a systemic cause
  7. lessons        shareable takeaways

blameless test: does any line blame a person for a system's failure?
  "engineer ran the wrong command"  -> "the process had no confirmation/guardrail
                                        that would have caught the wrong command"

action-item test: is it specific, owned, dated, and systemic?
  BAD:  "be more careful with deploys"
  GOOD: "add a required approval + dry-run to the deploy pipeline — @owner, by DATE"
```

### Reading a draft

- **Action items like "be more careful"** = not real fixes; they'll never be verified and change nothing. Replace with systemic, owned, dated changes.
- **A single "root cause" pinned on a person** = usually a missed systemic cause and a blameless failure; look for why the system permitted the impact.
- **No owners or dates on actions** = the postmortem will sit unimplemented, and the incident recurs. Every action needs both.
- **Only failures, no successes** = you'll erode the things that worked; capture what went well too.
- **Language naming culprits** = it teaches people to hide detail next time, degrading every future postmortem. Keep it role- and system-focused.

### Pitfalls

- **Blame culture.** The fastest way to make postmortems useless — people withhold the truth, and you fix symptoms instead of causes. Keep it genuinely blameless.
- **Vague action items.** Unowned, undated, or "try harder" actions don't happen. Specific + owner + date + systemic.
- **Chasing a single root cause.** Real incidents have multiple contributing factors; forcing one oversimplifies and under-fixes.
- **Writing it and filing it.** The value is in implementing the actions; an untracked postmortem is theatre. Follow up to completion.

### References

- Google SRE Book — Postmortem Culture: Learning from Failure
- NIST SP 800-61r2 (Post-Incident Activity)
- Etsy / blameless postmortem practices
- SANS Incident Handler's Handbook (lessons-learned phase)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.