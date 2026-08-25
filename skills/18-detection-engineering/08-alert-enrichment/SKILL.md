---
format: "v2"
name: "alert-enrichment"
title: "Alert Enrichment"
title_fr: "Enrichissement des alertes"
description: "Use when detections fire with too little context — adding asset, identity, and threat data automatically so analysts triage faster and with better decisions."
description_fr: "À utiliser quand les détections se déclenchent avec trop peu de contexte — ajouter automatiquement des données d'actif, d'identité et de renseignement sur la menace pour que les analystes trient plus vite et décident mieux."
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

A bare alert ("suspicious process on HOST-1234, user jsmith") forces the analyst to go hunt for context before they can even judge it — is HOST-1234 a domain controller or a test box, is jsmith an admin or an intern, is that IP known-bad? Alert enrichment adds that context *automatically*, so the alert arrives ready to triage. This skill covers enriching detections with the data that turns a raw signal into an actionable one, cutting triage time and improving decisions. It's how detections become usable, not just accurate.

### When to use it

When detections fire but analysts spend most of their triage time gathering context manually (a sign the SOC alert-triage workflow is slow because alerts are bare). It's a force multiplier — the same detections become far more efficient to work when enriched.

### Procedure

1. **Identify the context an analyst needs to triage — then automate gathering it.** For each alert, what does the analyst look up by hand? Usually: what/who is involved (asset and identity), is it known-bad (threat intel), and what's the surrounding activity. Enrichment pre-fetches exactly that.
2. **Enrich with asset context.** Attach the asset's criticality, environment (prod/dev), owner, and role from the inventory. "Suspicious process on a domain controller" is a very different alert from the same on a test VM — asset context changes the priority immediately.
3. **Enrich with identity context.** Attach the user's role, privilege level, and department. An admin account doing something unusual outranks a standard user doing the same; identity context shapes the severity.
4. **Enrich with threat intelligence.** Automatically look up IPs, domains, and hashes in the alert against threat-intel sources — a destination that's known-bad turns a maybe into a confirmed lead (feeds from the threat-intel domain). Add reputation and any known-actor association.
5. **Enrich with related activity.** Pull the surrounding events (what else did this host/user do around this time) so the analyst sees the alert in context rather than in isolation.
6. **Automate it with SOAR or pipeline enrichment.** The enrichment should happen automatically as the alert is created/ingested (SOAR playbooks, or enrichment in the log pipeline), not be a manual analyst step — automation is the whole point.
7. **Prefer enrichment over suppression for valid-but-noisy alerts.** Sometimes an alert is legitimate but hard to triage; enriching it (so the analyst decides in seconds) beats suppressing it (and losing the coverage) — this ties into the false-positive skill.

### Cheatsheet

```
bare alert = analyst hunts context before they can judge it -> slow triage
enrichment = pre-fetch that context AUTOMATICALLY -> alert arrives triage-ready

enrich with
  ASSET       criticality, env (prod/dev), owner, role  (DC vs test box = different alert)
  IDENTITY    role, privilege, department  (admin vs intern doing the same = different)
  THREAT INTEL  IP/domain/hash reputation + actor association (maybe -> confirmed lead)
  RELATED ACTIVITY  surrounding events for that host/user (alert in context, not isolation)

automate: SOAR playbooks / pipeline enrichment at alert creation (NOT a manual step)

use it well
  cuts triage TIME + improves decisions (context = better verdicts)
  prefer enrichment OVER suppression for valid-but-noisy alerts (keep coverage)
```

### Reading enrichment

- **Analysts spending most of triage time gathering context by hand** = the problem enrichment solves; the detections may be fine, but bare alerts make them slow to work. Automate the lookups the analyst does manually.
- **Asset context changing the priority** (the same alert on a DC vs a test box) = why asset enrichment matters; without it, a critical-asset alert looks identical to a trivial one and gets the same slow triage.
- **Identity context reshaping severity** (an admin account vs a standard user) = privilege makes the same behaviour more or less concerning; enrichment surfaces it up front.
- **A threat-intel hit on an alert's IP/domain** = turns an ambiguous signal into a confirmed lead automatically; the analyst starts from "known-bad" instead of "let me check". High-value enrichment.
- **A valid alert that's slow to triage** = a candidate for enrichment rather than suppression — keep the coverage, add the context so the verdict takes seconds.
- **Alerts arriving with asset, identity, intel, and related activity attached** = triage-ready detections; the SOC works them fast and decides well.

### Pitfalls

- **Leaving enrichment as a manual step.** If analysts still look everything up by hand, you haven't enriched — you've just documented what to look up. Automate it (SOAR/pipeline) at alert creation.
- **Enriching with noise.** Piling on irrelevant data buries the useful context; enrich with what actually informs the triage decision (asset, identity, intel, related activity), not everything available.
- **Missing asset/identity context.** These change priority the most and are often the missing piece; an alert without "how important is this asset/account?" forces the analyst to find out.
- **Suppressing valid-but-noisy alerts instead of enriching them.** Suppression loses coverage; for legitimate alerts that are just hard to triage, enrichment keeps the detection and cuts the effort.
- **Enriching without acting on it.** Context is only useful if it feeds the triage decision; wire enriched fields into the triage workflow and prioritisation.

### References

- SOAR platform documentation (enrichment playbooks)
- The SOC alert-triage-workflow and soar-automation skills, the vuln asset-inventory skill
- The threat-intelligence domain (intel enrichment sources)
- The reducing-false-positives skill (enrichment vs suppression)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.