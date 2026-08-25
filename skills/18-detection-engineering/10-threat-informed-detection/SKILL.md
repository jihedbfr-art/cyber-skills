---
format: "v2"
name: "threat-informed-detection"
title: "Threat Informed Detection"
title_fr: "Détection pilotée par la menace"
description: "Use when deciding which detections to build — driving the priorities from real threat intelligence about what attackers targeting you actually do, not from guesses or convenience."
description_fr: "À utiliser pour décider quelles détections construire — fonder les priorités sur du renseignement réel sur les techniques que les attaquants qui vous ciblent utilisent vraiment, plutôt que sur des suppositions ou la facilité."
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

You can't detect everything, so *what* you build detections for is a strategic choice — and the best guide is what real adversaries who target organisations like yours actually do. Threat-informed detection means letting threat intelligence drive the detection backlog: prioritise the techniques used by the actors and campaigns relevant to your sector and environment. This skill covers building detections from threat reality rather than from guesses, convenience, or whatever's easiest to write — the difference between a detection programme aimed at your threats and one aimed at nothing in particular.

### When to use it

When prioritising the detection backlog (which is always — there's more to detect than time to detect it). It sits on top of coverage mapping: mapping shows the gaps, threat intel shows which gaps *matter*, and together they order the work. It's what makes a detection programme strategic.

### Procedure

1. **Understand your threat model — who actually targets you.** Which threat actors, campaigns, and techniques are relevant to your sector, size, geography, and technology? A financial institution, a hospital, and a SaaS startup face different adversaries; a detection programme should reflect that, not detect generically.
2. **Source the intelligence.** Use threat intelligence (the threat-intel domain) — reporting on actors targeting your sector, ISAC feeds, and frameworks that track which techniques specific groups use. The MITRE ATT&CK groups/software data links actors to techniques, letting you derive "what would a relevant actor use here".
3. **Prioritise detections by relevant-threat techniques.** Cross the coverage map (what you can't detect — the mapping skill) with the threat intel (what relevant actors use). The intersection — techniques that are both uncovered and used by adversaries targeting you — is the top of the backlog. This is the core move.
4. **Weight by prevalence and impact.** Among relevant techniques, prioritise the ones that are widely used (many actors, high frequency) and high-impact — common initial-access and credential-access techniques often matter more than exotic ones only one actor uses.
5. **Update as the threat landscape shifts.** Adversary tradecraft evolves; a new campaign or a shift in actor behaviour changes what's worth detecting. Threat-informed detection is continuous — feed new intel into the backlog, and revisit priorities as threats change.
6. **Validate against real tradecraft.** Test the resulting detections against how the technique is actually used by the relevant actors (adversary emulation, purple teaming) so you're detecting real-world behaviour, not a textbook version.
7. **Close the loop with hunting and IR.** Threats found in your own environment (from hunts and incidents) are the most relevant intel of all — feed what actually hit you back into detection priorities.

### Cheatsheet

```
you can't detect everything -> WHAT to build is strategic
  best guide: what real adversaries who TARGET YOU actually do
  (not guesses / convenience / whatever's easiest to write)

1. threat MODEL: who targets you? (sector, size, geo, tech -> different adversaries)
2. SOURCE intel: sector reporting, ISACs, ATT&CK groups/software (actor->technique)
3. PRIORITISE: coverage gaps  ∩  relevant-actor techniques  = top of backlog
   (uncovered AND used against you — the core move)
4. weight by PREVALENCE + IMPACT (common initial-access/cred-access > exotic one-actor)
5. UPDATE continuously (tradecraft evolves ; new campaigns -> new priorities)
6. VALIDATE vs real tradecraft (emulation/purple team, not textbook version)
7. loop: your own hunts + incidents = the most relevant intel -> back into priorities
```

### Reading the priorities

- **A detection backlog driven by convenience** (whatever's easy to write, or the last blog post) = aimed at nothing in particular; it may leave the techniques actually used against you uncovered while detecting irrelevant ones. Reorient around your threat model.
- **The intersection of coverage gaps and relevant-actor techniques** = the highest-value detections to build; uncovered *and* used against you is precisely where effort pays off. This intersection is the whole point.
- **Prioritising exotic techniques over common ones** = often misallocated effort; widely-used initial-access and credential-access techniques catch more real attacks than a rare one-actor method. Weight by prevalence and impact.
- **Detections built for textbook technique versions** = may miss how the actor actually does it; validate against real tradecraft so the detection matches reality.
- **Threat intel from your own incidents/hunts not fed back** = ignoring the most relevant signal of all; what actually hit you is the strongest guide to what to detect next.
- **A backlog ordered by relevant-threat, prevalence-weighted, continuously updated** = a strategic, threat-informed programme aimed at your real adversaries.

### Pitfalls

- **Building detections by convenience or trend.** Writing rules for whatever's easy or currently hyped, rather than what targets you, produces a programme aimed at nothing in particular. Let threat intel drive priorities.
- **Ignoring your specific threat model.** Detecting generically wastes effort on irrelevant techniques while missing the ones adversaries actually use against your sector. Know who targets you.
- **Coverage without threat prioritisation.** The coverage map shows gaps but not which matter; without threat intel you may fill low-relevance gaps first. Cross the two.
- **Chasing exotic techniques.** Rare, single-actor methods are seductive but catch few real attacks; common initial-access/credential-access techniques usually matter more.
- **Static priorities.** Tradecraft evolves; a backlog set once and never revisited detects last year's threats. Update continuously, especially from your own incidents.

### References

- MITRE ATT&CK (groups and software — actor-to-technique mapping)
- The threat-intelligence domain, and the mapping-to-attack, testing-detections skills
- The red-team purple-teaming and threat-hunting skills (validation and feedback)
- Sector ISAC feeds and threat reporting

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.