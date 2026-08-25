---
format: "v2"
name: "hunting-with-attack"
title: "Hunting With Attack"
title_fr: "Chasse aux menaces guidée par MITRE ATT&CK"
description: "Use when using MITRE ATT&CK to structure and prioritise threat hunts — turning the framework into concrete, testable hunts instead of a poster on the wall."
description_fr: "À utiliser pour structurer et prioriser les chasses aux menaces avec MITRE ATT&CK : transformer le framework en chasses concrètes et testables plutôt qu'en simple poster décoratif."
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

MITRE ATT&CK catalogues how attackers operate, which makes it a natural map for deciding *what* to hunt for — but it's easy to treat it as a wall poster rather than a working tool. Hunting with ATT&CK means turning techniques into concrete hunts, prioritised by what's relevant to your environment, and tracking coverage so hunts build on each other. This skill covers using the framework to drive a hunting programme instead of hunting at random.

### When to use it

Planning what to hunt (the hardest part of hunting is choosing targets), and structuring a hunt programme so it's systematic and coverage-aware. It builds on the hypothesis-driven-hunting method by giving it a source of good, prioritised hypotheses.

### Procedure

1. **Pick techniques by relevance, not the whole matrix.** ATT&CK has hundreds of techniques; you can't hunt them all. Prioritise by your threat model (what actors targeting your sector use — the threat-intel domain), your environment (techniques that apply to your platforms), and gaps in your detection coverage (hunt what you can't already detect automatically). Relevance is the filter.
2. **Turn each technique into a hypothesis.** A technique isn't a hunt until it's a testable statement: "if an attacker used T1055 (process injection) here, I'd see [specific telemetry signature] in [this data source]." This bridges ATT&CK to the hypothesis-driven method.
3. **Confirm you can see it.** Each technique needs specific telemetry (ATT&CK data sources map this); if you don't collect the data, the hunt can't conclude — and that's a visibility gap finding (the log-source-coverage skill).
4. **Hunt the technique**, investigate hits, and conclude — threat found, clean, or visibility gap (the hypothesis-driven-hunting method applies).
5. **Track coverage on the ATT&CK matrix.** Record which techniques you've hunted and the outcome, using the ATT&CK Navigator, so hunts accumulate into a coverage map. This prevents re-hunting the same thing and shows where the gaps are.
6. **Prioritise by prevalence and impact.** Among relevant techniques, hunt the common, high-impact ones first (widely-used initial-access, credential-access, lateral-movement techniques) — they catch more real activity than exotic ones.
7. **Feed results back.** A hunt that repeatedly finds something becomes a detection (the operationalising skill); a hunt that exposes a visibility gap becomes a telemetry request. ATT&CK-structured hunting continuously improves both detection coverage and visibility.

### Cheatsheet

```
ATT&CK = the map of HOW attackers operate -> what to hunt (not a wall poster)

1. PICK by relevance (not the whole matrix — hundreds of techniques)
     threat model (actors targeting you) + your environment + detection gaps
2. TECHNIQUE -> HYPOTHESIS (testable)
     "if T#### used here, I'd see [signature] in [data source]"
3. CAN YOU SEE IT? (ATT&CK data sources) — no data = visibility gap finding
4. HUNT -> investigate -> conclude (found / clean / visibility gap)
5. TRACK on ATT&CK Navigator (hunted + outcome) -> coverage map, no re-hunting
6. PRIORITISE prevalence + impact (common initial-access/cred/lateral > exotic)
7. FEED BACK: repeatable find -> detection ; visibility gap -> telemetry request
```

### Reading the approach

- **Hunting random techniques off the matrix** = unfocused; you may hunt irrelevant ones while the techniques attackers actually use against you go unhunted. Prioritise by relevance — threat model, environment, coverage gaps.
- **A technique treated as a hunt without a hypothesis** = not actionable; "hunt for T1055" isn't a hunt until it's "I'd see X in Y". Turn it into a testable statement.
- **A hunt for a technique you can't see** (no telemetry) = it can't conclude; recognise the visibility gap as the finding and route it to telemetry.
- **No coverage tracking** = you re-hunt the same techniques and can't see gaps; the Navigator map makes hunts cumulative and strategic.
- **Hunting exotic techniques first** = often lower yield; common high-impact techniques catch more real activity. Prioritise by prevalence and impact.
- **ATT&CK-prioritised, hypothesis-framed, coverage-tracked hunting feeding detection** = a systematic programme, not random hunting.

### Pitfalls

- **Treating ATT&CK as a poster.** The framework's value is driving concrete, prioritised hunts; admiring the matrix without turning techniques into hypotheses achieves nothing.
- **Trying to hunt the whole matrix.** Hundreds of techniques, finite time — hunting all is impossible and unfocused. Prioritise ruthlessly by relevance.
- **Skipping the visibility check.** Hunting a technique you can't see wastes effort; confirm the data source exists (and treat its absence as a finding).
- **No coverage tracking.** Without recording what's been hunted, you repeat work and can't identify gaps; use the Navigator to make hunts cumulative.
- **Ignoring prevalence.** Chasing rare techniques over common high-impact ones lowers yield; hunt what attackers actually do most.

### References

- MITRE ATT&CK and the ATT&CK Navigator; ATT&CK Data Sources
- The hypothesis-driven-hunting, operationalising-a-hunt, and detection mapping-to-attack skills
- The threat-intelligence domain (relevance prioritisation)
- MITRE's threat-hunting and coverage guidance

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.