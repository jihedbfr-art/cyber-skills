---
format: "v2"
name: "purple-teaming"
title: "Purple Teaming"
title_fr: "Purple teaming"
description: "Use when running red and blue collaboratively — red executes techniques while blue watches and tunes detection in real time, so the engagement directly improves defensive coverage."
description_fr: "À utiliser pour faire travailler les équipes rouge et bleue de façon collaborative — l'équipe rouge exécute des techniques pendant que l'équipe bleue observe et ajuste la détection en temps réel, afin que l'engagement améliore directement la couverture défensive."
domain: "16-red-teaming-and-adversary-emulation"
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

A traditional red-team engagement is adversarial and often produces a report weeks later; purple teaming makes it collaborative and immediate — the red side executes techniques while the blue side watches, and together they confirm what's detected, tune what isn't, and re-test on the spot. The result is directly improved detection coverage rather than just a list of what got through. This skill covers running purple-team exercises, the most efficient way to turn offensive testing into defensive improvement.

### When to use it

When the goal is improving detection and response (not just testing if you can break in). Purple teaming is ideal for building and validating detection coverage against specific techniques, and it's a natural collaboration between red team, detection engineering, threat hunting, and the SOC.

### Procedure

1. **Set up the collaboration.** Red and blue work together, not against each other — often in the same room or call. The red side plans to execute specific techniques (from the emulation plan / ATT&CK), and the blue side is ready to observe their telemetry and detections. This transparency is the difference from red teaming.
2. **Execute techniques deliberately and one at a time.** Rather than a stealthy end-to-end intrusion, purple teaming runs techniques methodically — often using atomic tests (Atomic Red Team) mapped to ATT&CK — so the blue side can observe exactly what each technique looks like in their telemetry. Deliberate, technique-by-technique execution is the method.
3. **Confirm detection in real time — the core loop.** For each technique: did the blue side detect it? If yes, confirm the detection works and is trustworthy. If no, that's a gap — and because red and blue are collaborating, they can immediately investigate why (missing telemetry? no rule? a rule that didn't fire?) and *fix it on the spot*. This immediate detect → tune → re-test loop is what makes purple teaming efficient.
4. **Tune and build detections live.** When a technique isn't detected, the blue side builds or fixes a detection (the detection-engineering skills) and the red side re-runs the technique to confirm it now fires. You leave the exercise with improved coverage, not just a finding to fix later.
5. **Map coverage to ATT&CK.** Track which techniques are detected and which aren't on the ATT&CK matrix (the detection mapping skill), producing a concrete coverage map from the exercise — and prioritising which gaps to close.
6. **Validate telemetry too.** A technique undetected because of *missing telemetry* (not a missing rule) is a different fix (add logging — the log-source-coverage skill); purple teaming reveals which gaps are visibility vs detection.
7. **Document coverage gained and remaining.** The output is the improved detection coverage, the detections built/tuned, and the prioritised remaining gaps — a far more actionable result than a traditional red-team report.

### Cheatsheet

```
red team = adversarial + report weeks later ; PURPLE = collaborative + IMMEDIATE
  red executes + blue watches -> confirm detected, TUNE what isn't, RE-TEST on the spot
  -> directly improves detection COVERAGE (not just "what got through")

run it
  1. COLLABORATE (same room/call ; transparency — the difference from red teaming)
  2. execute DELIBERATELY, one technique at a time (Atomic Red Team, ATT&CK-mapped)
       -> blue observes exactly what each looks like in telemetry
  3. CORE LOOP: detected? yes -> confirm works | no -> investigate WHY + FIX ON THE SPOT
       (missing telemetry? no rule? rule didn't fire?) -> detect->tune->re-test
  4. build/tune detections LIVE -> red re-runs -> confirm it fires now
       (leave with improved COVERAGE, not a finding for later)
  5. map coverage to ATT&CK (concrete coverage map + prioritise gaps)
  6. VISIBILITY vs DETECTION gap (missing telemetry = add logging, not a rule)
  output: coverage gained + detections built + prioritised remaining gaps (>> a red-team report)
```

### Reading the exercise

- **The detect → tune → re-test loop closing on a technique** (undetected → detection built → now fires) = purple teaming working; you leave with new coverage confirmed, not just a documented gap. This immediate improvement is the whole value.
- **A technique undetected** = a coverage gap found collaboratively; because red and blue are together, they investigate and fix it immediately rather than reporting it for weeks-later remediation.
- **An undetected technique due to missing telemetry (not a missing rule)** = a visibility gap, a different fix (add logging — the log-source-coverage skill). Purple teaming distinguishes visibility gaps from detection gaps.
- **A detection confirmed to fire on the real technique** = validated coverage (the testing-detections skill); you know it works, not just that a rule exists.
- **A coverage map from the exercise** = concrete, ATT&CK-mapped evidence of what's detected and what isn't, prioritising the remaining work.
- **Improved coverage and built detections as the output** = a far more actionable result than a traditional red-team "here's what got through" report; the exercise directly advanced the defence.

### Pitfalls

- **Running it adversarially (like a red team).** Purple teaming's value is transparency and immediate collaboration; a stealthy, adversarial approach loses the detect → tune → re-test loop. Work together, openly.
- **Not fixing gaps on the spot.** The efficiency comes from tuning and re-testing immediately; deferring fixes to a later report throws away the main advantage. Build detections live.
- **Skipping re-test.** Building a detection isn't enough; re-run the technique to confirm it actually fires (validation). Untested new detections are hypotheses.
- **Confusing visibility and detection gaps.** An undetected technique may need telemetry, not a rule; distinguish them or you'll write rules for data you don't collect.
- **Not mapping to ATT&CK.** Without the coverage map, the exercise's results aren't trackable or prioritisable; map what's detected and what isn't.

### References

- Atomic Red Team (per-technique tests), MITRE Caldera, and the ATT&CK Navigator
- The detection-engineering domain (writing/testing detections, mapping-to-attack, log-source-coverage) — the blue side
- The threat-hunting and SOC domains, and the attack-emulation-planning skill
- MITRE and industry purple-teaming methodologies

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.