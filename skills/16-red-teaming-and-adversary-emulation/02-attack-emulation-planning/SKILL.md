---
name: attack-emulation-planning
domain: 16-red-teaming-and-adversary-emulation
description: Use when planning an adversary emulation — mapping a real threat actor's behaviour to a scenario grounded in MITRE ATT&CK, so the engagement tests defences against threats that actually matter.
difficulty: intermediate
tags: [red-team, emulation, attack, planning, scenario]
tools: [attack-navigator, caldera]
---

## Purpose

The difference between "run some attacks" and adversary *emulation* is planning: emulation replicates how a specific, relevant threat actor operates, so the engagement tests the organisation's defences against threats it actually faces. This skill covers planning an emulation — choosing a relevant adversary, mapping their tradecraft to a scenario grounded in MITRE ATT&CK, and defining objectives — so the offensive work produces meaningful defensive insight rather than a scattershot of techniques.

## When to use it

After scoping/RoE (that skill — always first), when planning what the engagement will actually do. Good planning is what makes a red-team engagement valuable to the blue team; a plan grounded in a real threat produces findings the organisation can act on.

## Procedure

1. **Choose a relevant adversary — grounded in the organisation's threat model.** Emulate a threat actor that actually targets your sector/organisation (from threat intelligence — the threat-intel domain), not a generic or arbitrary one. Emulating an actor irrelevant to the organisation tests against a threat they don't face. Relevance is what makes the engagement meaningful.
2. **Map the adversary's TTPs to ATT&CK.** Threat intelligence and ATT&CK's group profiles describe how the actor operates — their techniques across the kill chain (initial access, execution, persistence, lateral movement, exfiltration). Build the scenario from their real tradecraft, so you're emulating *this actor*, not improvising.
3. **Define clear objectives.** What is the emulation trying to achieve — reach specific data/systems (a "flag"), test detection of specific techniques, validate a control? Goal-based objectives (reach the crown jewels) test the whole chain; technique-based objectives (exercise these ATT&CK techniques) test specific coverage. Define which.
4. **Plan the scenario as a sequence.** Lay out the emulated attack path step by step — how the actor gains access, escalates, moves, and achieves the objective — mapped to ATT&CK techniques. This sequence is the plan the engagement executes and the blue team can compare their detections against.
5. **Decide the engagement model** (with scoping): assumed-breach (start with a foothold, focus on post-exploitation) vs full-scope (start from outside); how much the blue team knows (announced vs unannounced); and whether it's a red-team (adversarial) or purple-team (collaborative) exercise.
6. **Align with the blue team's needs.** The point is improving defence, so plan to test what the organisation needs to know about its detection and response — coordinate with detection engineering on which techniques and coverage gaps to exercise (ties into threat-informed detection).
7. **Document the plan** — objectives, the emulated actor, the ATT&CK-mapped scenario, and success criteria — so the engagement is structured and its results are comparable to the plan.

## Cheatsheet

```
emulation (not just "run attacks") = replicate a SPECIFIC RELEVANT actor's behaviour
  -> tests defences against threats you ACTUALLY face -> meaningful insight

plan (after scoping/RoE)
  1. RELEVANT adversary (threat model / threat-intel — actor that targets YOUR sector)
       not generic/arbitrary (irrelevant actor = tests a threat you don't face)
  2. map their TTPs to ATT&CK (group profiles + intel -> real tradecraft across kill chain)
  3. OBJECTIVES: goal-based (reach the crown jewels — whole chain) vs technique-based (coverage)
  4. scenario as a SEQUENCE (access->escalate->move->objective, ATT&CK-mapped)
       = the plan to execute + what blue team compares detections against
  5. engagement model: assumed-breach vs full-scope ; announced vs not ; red vs PURPLE
  6. align with BLUE TEAM needs (test detection/coverage gaps — threat-informed detection)
  7. DOCUMENT (objectives, actor, ATT&CK scenario, success criteria)
```

## Reading the plan

- **A plan grounded in a relevant threat actor** = the engagement tests defences against threats the organisation actually faces; the findings are directly actionable. This relevance is what separates emulation from scattershot attacking.
- **A generic/arbitrary "run some attacks" plan** = tests against threats that may not matter to the organisation; the results are less actionable. Ground it in the real threat model.
- **ATT&CK-mapped scenario** = comparable to the blue team's detection coverage; they can see exactly which techniques were used and which they caught. This mapping is what makes the exercise useful to defenders.
- **Clear objectives** (goal-based or technique-based) = a measurable engagement; without defined objectives, "success" is subjective and the results are hard to act on.
- **Alignment with blue-team detection needs** = the engagement tests what the organisation needs to learn; a red team operating in a vacuum produces less defensive value.
- **A documented, actor-grounded, ATT&CK-mapped, objective-driven plan** = an emulation that produces meaningful defensive insight.

## Pitfalls

- **"Running attacks" instead of emulating an actor.** Scattershot techniques test against no particular threat; emulate a specific, relevant adversary so the engagement tests defences that matter.
- **Choosing an irrelevant adversary.** Emulating an actor that doesn't target your sector tests a threat you don't face; ground the choice in your threat model and intel.
- **No ATT&CK mapping.** Without it, the blue team can't compare their coverage to what you did; mapping makes the exercise comparable and useful.
- **Undefined objectives.** Without clear goals, engagement success is subjective and results are hard to act on. Define goal- or technique-based objectives.
- **Operating in isolation from the blue team.** The point is improving defence; plan to test the detection and coverage the organisation needs to understand (especially in purple-team mode).

## References

- MITRE ATT&CK, ATT&CK Navigator, and ATT&CK group/software profiles
- MITRE Caldera and adversary-emulation-plan resources (e.g. CTID emulation plans)
- The scoping-and-rules-of-engagement, purple-teaming skills and the threat-intelligence domain
- The detection threat-informed-detection skill (align on coverage)
