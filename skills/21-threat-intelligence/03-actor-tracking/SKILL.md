---
name: actor-tracking
domain: 21-threat-intelligence
description: Use when tracking a threat actor over time — attributing activity, following their evolving tradecraft, and turning "who and how" into defensive advantage.
difficulty: advanced
tags: [threat-intel, actors, attribution, tradecraft, tracking]
tools: [misp]
---

## Purpose

Beyond individual indicators lies the actor — the group behind the activity, with characteristic tools, techniques, targets, and infrastructure. Tracking an actor over time lets you anticipate their moves, recognise their activity from behaviour even when indicators change, and prioritise defences against the adversaries who actually target you. This skill covers following threat actors and turning that knowledge into defence — higher up the intelligence value chain than IoCs, and far more durable.

## When to use it

When your programme matures beyond indicator management to understanding adversaries, and when specific actors are relevant to your organisation (your sector is targeted by known groups). It connects to the Pyramid of Pain — tracking an actor's TTPs is the most painful thing you can do to them.

## Procedure

1. **Understand what defines an actor.** A threat actor (APT group, criminal crew) is characterised by their **TTPs** (how they operate), **tools** (custom and commodity), **infrastructure** (C2 patterns, hosting preferences), **targets** (sectors, geographies), and **motivation** (espionage, financial, hacktivism). This profile, not any single indicator, is the actor.
2. **Track by behaviour, not just indicators.** Infrastructure and hashes change; TTPs are far more stable. Following an actor means tracking *how they operate* — their characteristic techniques and tradecraft — so you recognise them even on new infrastructure. This behavioural tracking is what makes actor intelligence durable (Pyramid of Pain).
3. **Handle attribution carefully — it's hard and often uncertain.** Linking activity to a specific actor is difficult: actors share tools, deliberately plant false flags, and evolve. Attribution should carry a confidence level and be based on a preponderance of evidence (TTPs, infrastructure overlaps, targeting, timing), not a single indicator. Overconfident attribution misleads decisions.
4. **Map actor activity to ATT&CK.** Expressing an actor's known techniques in ATT&CK terms makes them comparable, feeds threat-informed detection (which actors use which techniques), and lets you assess your coverage against a specific adversary.
5. **Follow evolution over time.** Actors adapt — new tools, changed infrastructure, shifted targeting. Tracking is continuous; a profile built once goes stale as the actor evolves. Update as new reporting and your own observations come in.
6. **Prioritise actors relevant to you.** You can't track every group; focus on the actors known to target your sector, geography, and technology (from sector reporting and ISACs). Tracking an irrelevant actor is effort spent on a threat you don't face.
7. **Turn tracking into defence.** The payoff: use actor knowledge to prioritise detections for their techniques (threat-informed detection), anticipate their next moves, hunt for their tradecraft, and brief leadership on the specific threats you face (strategic intel).

## Cheatsheet

```
above IoCs = the ACTOR (the group behind the activity) — durable, higher-value

an actor is defined by (not any single indicator)
  TTPs (how they operate — most stable) | tools | infrastructure patterns
  | targets (sector/geo) | motivation

track by BEHAVIOUR not indicators (infra/hashes change ; TTPs stable)
  -> recognise them on new infrastructure (Pyramid of Pain: TTPs = most painful)

ATTRIBUTION is hard: shared tools, FALSE FLAGS, evolution
  -> confidence level + preponderance of evidence, NOT one indicator (overconfidence misleads)

map to ATT&CK (comparable + feeds threat-informed detection + coverage-vs-actor)
follow EVOLUTION (continuous ; profiles go stale)
PRIORITISE actors relevant to YOU (sector/geo/tech) — can't track everyone

payoff -> DEFENCE: prioritise detections for their techniques, anticipate, hunt, brief leadership
```

## Reading actor intelligence

- **An actor's TTP profile** (their characteristic techniques and tradecraft) = the durable, high-value intelligence; because TTPs are stable while infrastructure rotates, tracking behaviour lets you recognise the actor even when their indicators are all new. This is the point of actor tracking.
- **Attribution based on a single indicator** = unreliable; actors share tools and plant false flags. Attribution needs a preponderance of evidence and a stated confidence level — overconfident attribution misleads decisions and can even be the adversary's goal (false flags).
- **An actor mapped to ATT&CK techniques** = you can assess your detection coverage against that specific adversary and prioritise accordingly (threat-informed detection). A concrete defensive output.
- **A stale actor profile** = actors evolve (new tools, infrastructure, targeting); an out-of-date profile tracks who they *were*. Update continuously.
- **Tracking an actor irrelevant to your sector** = effort on a threat you don't face; prioritise the groups actually targeting your organisation.
- **Actor knowledge driving detection priorities, hunts, and leadership briefings** = tracking turned into defence — the reason it's worth doing.

## Pitfalls

- **Tracking indicators instead of behaviour.** Infrastructure and hashes change constantly; following an actor by TTPs is what stays valid across their infrastructure changes. Track behaviour.
- **Overconfident attribution.** Actors share tools and plant false flags deliberately; single-indicator attribution is unreliable and can be exactly what the adversary wants you to conclude. Use confidence levels and multiple evidence types.
- **Static profiles.** Actors evolve; a profile built once and never updated tracks the past. Follow their evolution continuously.
- **Tracking irrelevant actors.** You can't follow every group; effort on actors that don't target your sector is wasted. Prioritise by relevance.
- **Tracking without operationalising.** Actor knowledge that doesn't drive detection priorities, hunts, and defensive decisions is trivia; turn it into defence.

## References

- MITRE ATT&CK Groups (actor-to-technique profiles) and the Pyramid of Pain
- The threat-informed-detection, mapping-intel-to-detection, and tactical-vs-strategic skills
- Diamond Model of Intrusion Analysis (adversary/infrastructure/capability/victim)
- Sector ISAC reporting and vendor actor-tracking research
