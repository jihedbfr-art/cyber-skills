---
name: mapping-to-attack
domain: 18-detection-engineering
description: Use when tying detections to the MITRE ATT&CK framework — mapping each rule to a technique so you can measure coverage, find gaps, and prioritise what to build next.
difficulty: intermediate
tags: [detection, attack, coverage, mitre, prioritisation]
tools: [attack-navigator]
---

## Purpose

Writing detections without a framework leaves you unable to answer the most important question: *what can't we detect?* Mapping every detection to MITRE ATT&CK — the standard catalogue of attacker tactics and techniques — turns your rule set into a coverage map. You can see which techniques you catch, where the gaps are, and build the next rule where it matters instead of adding another detection for something already covered. This skill covers mapping to ATT&CK and using the coverage view to drive detection priorities.

## When to use it

Continuously as you build detections (tag each with its technique) and periodically to assess and prioritise coverage. It's what makes a detection programme strategic rather than a random accumulation of rules, and it feeds metrics, threat-informed detection, and threat hunting.

## Procedure

1. **Tag every detection with its ATT&CK technique(s)** in the rule metadata (Sigma has a `tags` field for this). A rule detecting Kerberoasting maps to T1558.003; a rule for scheduled-task persistence to T1053.005. This tagging is the raw material for the whole coverage view.
2. **Build a coverage map.** Aggregate the technique tags across all detections onto the ATT&CK matrix — the ATT&CK Navigator visualises this, colouring techniques by whether (and how well) you detect them. Now "what do we cover?" is a picture, not a guess.
3. **Read the gaps.** Uncovered techniques are your blind spots. But don't treat all gaps equally — prioritise by which techniques are relevant to *your* threat model and environment (a technique that only applies to a platform you don't run is a lower-priority gap).
4. **Weight coverage by quality, not just presence.** Having "a detection" for a technique isn't the same as detecting it well — a narrow rule catching one variant leaves most of the technique uncovered. Be honest: partial coverage is partial, and the map should reflect confidence, not just a checkmark.
5. **Prioritise new detections by threat-informed gaps.** Combine the coverage map with threat intelligence (what actors targeting your sector actually use — the threat-informed-detection skill) to build detections for the techniques that are both uncovered and relevant. This is how mapping drives what you write next.
6. **Avoid coverage theatre.** A green matrix can be misleading if the detections are weak or untested. Coverage is a means (finding gaps) not an end (a pretty chart); pair it with detection testing so "covered" means "actually detects".
7. **Track coverage over time** as a programme metric and communicate it — it shows progress and justifies detection investment (feeds the metrics skill).

## Cheatsheet

```
tag every detection with its ATT&CK technique (Sigma `tags:` field)
  Kerberoasting -> T1558.003 ; scheduled task -> T1053.005 ; etc.

build the coverage map
  aggregate tags onto the ATT&CK matrix -> ATT&CK Navigator (colour by coverage)
  now "what do we cover / MISS?" is a picture

read gaps SMARTLY (not all equal)
  prioritise gaps by YOUR threat model + environment relevance
  (technique for a platform you don't run = low-priority gap)

weight by QUALITY not presence
  "a detection exists" != "detects the technique well"
  narrow rule = partial coverage -> reflect CONFIDENCE, not just a checkmark

drive what to build next: uncovered AND relevant (threat-informed) techniques
avoid coverage THEATRE: green matrix + weak/untested rules = false comfort
  -> pair with detection testing (covered = actually detects)
track over time -> programme metric
```

## Reading the coverage map

- **A clear gap in a technique relevant to your threat model** = a priority detection to build; the map's main value is surfacing exactly this. An uncovered technique that attackers targeting you actually use is where the next rule goes.
- **A "covered" technique with only a narrow, untested rule** = coverage theatre; the checkmark overstates reality. Weight by quality — partial coverage is partial, and untested coverage is unproven.
- **Broad green coverage that looks reassuring** = check whether it's real (tested, quality detections) or superficial; a pretty matrix can hide weak rules. Coverage is for finding gaps, not for the chart.
- **Gaps in techniques irrelevant to your environment** (a platform you don't run, an actor with no interest in your sector) = lower priority; not every gap deserves equal effort. Prioritise by relevance.
- **Duplicate detections for already-covered techniques** = effort that could have gone to a gap; the map prevents this by showing what's already handled.
- **A quality-weighted, threat-prioritised coverage map driving the backlog** = mapping working as intended — strategic detection development.

## Pitfalls

- **Not mapping at all.** Without ATT&CK tags you can't see coverage or gaps; the programme becomes a random pile of rules and you can't answer "what can't we detect?".
- **Treating presence as coverage.** A checkmark for "has a detection" hides weak, narrow, or untested rules. Weight by quality and confidence, not just existence.
- **Coverage theatre.** Optimising for a green matrix produces superficial detections; coverage is a tool for finding gaps, not a scoreboard. Pair with testing.
- **Treating all gaps equally.** Chasing every uncovered technique wastes effort on irrelevant ones; prioritise by your threat model and environment.
- **Mapping without threat intel.** Coverage tells you the gaps; threat intel tells you which matter. Combine them to prioritise (threat-informed detection).

## References

- MITRE ATT&CK and the ATT&CK Navigator
- Sigma rule `tags` (ATT&CK mapping convention)
- The writing-sigma-rules, testing-detections, threat-informed-detection, and detection-metrics skills
- MITRE's guidance on ATT&CK-based coverage assessment
