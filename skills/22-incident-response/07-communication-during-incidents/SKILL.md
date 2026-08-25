---
format: "v2"
name: "communication-during-incidents"
title: "Communication During Incidents"
title_fr: "Communication pendant un incident"
description: "Use during an incident to manage who gets told what, when — internal coordination, stakeholder updates, and external/legal notifications — without leaking or misinforming."
description_fr: "À utiliser pendant un incident pour gérer qui est informé, de quoi et quand — coordination interne, mises à jour aux parties prenantes et notifications externes/légales — sans fuite d'information ni désinformation."
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

Technical response and communication run in parallel during an incident, and poor communication sinks otherwise good response work — leaked details tip off the attacker, silence breeds panic, and a premature or wrong public statement creates legal and reputational damage. This skill covers who to tell, when, through what channel, and how much — the coordination that keeps an incident from becoming two crises.

### When to use it

From the moment an incident is declared. Communication planning belongs in the incident from the start, not bolted on when the press calls. The severity (from triage) drives how much of this applies.

### The audiences

- **The response team** — needs full, fast, accurate technical detail to coordinate.
- **Internal stakeholders** (leadership, legal, affected business units) — need enough to make decisions and meet obligations, framed for their role, not raw technical dumps.
- **Affected users/customers** — need honest, timely notice when their data or service is impacted, without speculation.
- **External parties** — regulators, law enforcement, insurers, partners — often on legally-defined timelines.
- **The public / press** — via a single controlled channel, only when appropriate.

### Procedure

1. **Establish a coordination channel** for the response team — and be aware it may be monitored if the attacker is still in your environment. For serious incidents, use an **out-of-band** channel (not the potentially-compromised corporate email/chat) so you're not coordinating where the attacker can read it.
2. **Designate a single point of communication** (an incident lead or comms lead). Mixed messages from multiple people cause confusion and contradictions; route external statements through one owner.
3. **Match detail to audience.** Give the response team everything; give leadership decisions and impact; give users clear, non-technical, honest information. Don't send raw indicators to people who can't use them, and don't over-share technical detail that could aid the attacker if it leaks.
4. **Say what you know, flag what you don't.** Early incidents are uncertain — communicate current facts, label assumptions as assumptions, and avoid speculation that you'll have to walk back. "We're investigating and will update by [time]" beats a confident guess.
5. **Involve legal early** for notification obligations. Data breaches carry regulatory timelines (e.g. GDPR 72 hours) and specific requirements — legal and compliance decide external notifications, not the technical team alone.
6. **Time external communication carefully.** Don't tip off an attacker who's still present, but don't unlawfully delay a required breach notification either — this is a legal/leadership judgement, made with counsel.
7. **Keep a communication log** — who was told what, when — as part of the incident record; it matters for the postmortem and for demonstrating you met obligations.

### Cheatsheet

```
audiences (detail level)
  response team    -> full technical detail, fast
  leadership/legal -> impact + decisions needed, role-framed
  users/customers  -> honest, plain-language, no speculation
  regulators/LE    -> per legal timelines (legal owns this)
  press/public     -> single controlled channel, only when appropriate

rules
  [ ] one designated comms owner (no mixed messages)
  [ ] out-of-band channel if the attacker may be watching corp comms
  [ ] say what's known, label assumptions, promise a next-update time
  [ ] legal in early (GDPR 72h etc. — notification is a legal decision)
  [ ] don't tip off a present attacker; don't unlawfully delay notice
  [ ] log every communication (who/what/when)
```

### Reading the situation

- **The attacker may still be in your systems** = coordinate out-of-band; discussing the response over compromised corporate channels hands them your playbook.
- **Data of users/regulated info involved** = legal notification timelines are likely triggered; get counsel in immediately — this is often the highest-stakes comms decision.
- **Pressure for an early public statement** = resist confident claims while facts are uncertain; a wrong statement is worse than "investigating, update at [time]".
- **Multiple people talking externally** = contradictions and leaks; funnel through one owner.
- **A quiet, unconfirmed incident** may not warrant broad communication yet — match the reach to the severity, but keep the log either way.

### Pitfalls

- **Coordinating over compromised channels.** If the attacker reads your incident chat, they counter every move. Go out-of-band for serious incidents.
- **Speculating early.** Confident wrong statements have to be retracted and erode trust. Communicate facts and label the rest.
- **Leaving legal out until late.** Notification obligations have deadlines; discovering them after the window is a compliance failure on top of the breach.
- **Multiple uncoordinated spokespeople.** Mixed messages confuse everyone and can leak sensitive detail. One owner.
- **No communication log.** You'll need to show who was notified and when — for the postmortem and for regulators.

### References

- NIST SP 800-61r2 (coordination and information sharing)
- GDPR Article 33/34 and equivalent breach-notification regimes (with counsel)
- SANS incident communication guidance
- FIRST — incident coordination resources

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.