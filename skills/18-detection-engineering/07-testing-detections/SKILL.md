---
name: testing-detections
domain: 18-detection-engineering
description: Use when validating that a detection actually works — running the technique it targets to confirm it fires, and checking it stays quiet on benign activity, before trusting it.
difficulty: intermediate
tags: [detection, testing, validation, atomic-red-team, emulation]
tools: [atomic-red-team, caldera]
---

## Purpose

An untested detection is a hope, not a control. A rule can be syntactically perfect, deployed, and mapped to ATT&CK — and still never fire on the actual attack, because of a wrong field, a telemetry gap, or flawed logic. Testing detections means running the technique the rule targets and confirming the alert fires (and that benign activity doesn't trigger it). This skill covers validating detections so "covered" means "actually detects", closing the gap between assumed and real coverage.

## When to use it

Before trusting any new detection, after tuning one (confirm it still fires), and periodically to catch detections that broke when the environment or telemetry changed. It's what makes coverage claims honest and turns detection-as-code's CI into real validation.

## Procedure

1. **Test that the detection fires on the real technique — the essential check.** Execute the attack technique the rule targets, safely, and confirm the alert triggers. Atomic Red Team provides small, safe, per-technique tests mapped to ATT&CK — run the atomic test for the technique and watch for your alert:
   ```
   # Atomic Red Team: run the atomic test for the technique (e.g. T1558.003)
   # -> confirm your detection fires
   ```
2. **Test in a safe environment.** Run technique tests in a lab or a controlled, authorised segment — some are benign, but you're deliberately generating malicious-looking activity, so scope it and coordinate (don't trigger a real IR by testing in prod unannounced).
3. **Confirm the negative case too.** A good test checks both: the rule fires on the malicious technique, *and* it stays quiet on the benign activity that could false-positive. A rule that fires on everything "passes" the positive test but fails in practice.
4. **Escalate to adversary emulation for chains.** Atomic tests validate individual techniques; for detecting an attack *sequence*, emulate a fuller adversary (Caldera, or a manual purple-team exercise) and confirm detections fire across the kill chain, not just in isolation. This also finds gaps between techniques.
5. **Automate testing in the pipeline where possible.** Detection-as-code CI can run technique tests against a detection on commit, so a rule that stops firing (a broken field, a schema change) is caught immediately rather than discovered during an incident.
6. **Purple-team it.** The strongest validation is red and blue together — the red side runs techniques, the blue side confirms detection and tunes what's missing. Testing detections is a natural collaboration point (feeds the red-team purple-teaming skill).
7. **Record results and re-test.** Track which detections are validated and when; detections rot as environments change, so validation is periodic, not one-time.

## Cheatsheet

```
untested detection = a HOPE, not a control.
  can be syntactically perfect + deployed + ATT&CK-mapped and STILL never fire.

essential test: run the technique -> does the alert fire?
  Atomic Red Team = small safe per-technique tests mapped to ATT&CK
    run the atomic for T#### -> confirm your detection triggers
  safe environment: lab / controlled authorised segment (don't spook prod IR)

test BOTH cases
  positive: fires on the malicious technique?
  negative: stays QUIET on the benign activity that could FP?  (fires-on-everything = fail)

chains: adversary emulation (Caldera / purple team) -> detections fire across the
        kill chain, not just isolated techniques (finds gaps between them)

automate in CI (detection-as-code): re-test on commit -> catch broken rules early
purple team: red runs techniques, blue confirms+tunes (strongest validation)
record + RE-TEST periodically (detections rot as environment changes)
```

## Reading the tests

- **A detection that doesn't fire when you run its technique** = the most important catch; the rule was assumed coverage you don't actually have. The cause (wrong field, telemetry gap, flawed logic) is now fixable — before an incident, not during. This is the whole point of testing.
- **A detection that fires on the atomic test** = validated positive coverage; you can now trust it catches that technique. Coverage claims for it are honest.
- **A rule that fires on the technique but also on benign activity** = passes the positive test but will flood the SOC; the negative test catches this. Both cases matter.
- **Individual techniques detected but the chain missed** = gaps between techniques; adversary emulation reveals that detections work in isolation but leave seams an attacker slips through. Atomic tests alone don't find this.
- **A previously-validated detection that now doesn't fire** = it rotted (a field renamed, a log source changed); periodic re-testing and CI catch the regression.
- **Detections validated by purple teaming with results tracked** = the mature state; coverage is proven, not assumed.

## Pitfalls

- **Trusting untested detections.** Syntactically valid and deployed doesn't mean it fires on the real attack; a wrong field or telemetry gap makes it silent. Run the technique and confirm.
- **Testing only the positive case.** A rule that fires on the technique but also on everything benign is useless in production; check it stays quiet on benign activity too.
- **Testing in production unannounced.** Generating malicious-looking activity can trigger a real IR or alarm the SOC; scope and coordinate the tests.
- **Only atomic-testing individual techniques.** Detecting each technique in isolation can still miss the attack chain; emulate sequences to find the gaps between them.
- **One-time validation.** Detections break silently as environments change; without periodic re-testing (ideally in CI), you rediscover the breakage during an incident.

## References

- Atomic Red Team (atomicredteam.io) — per-technique tests
- MITRE Caldera and adversary-emulation frameworks
- The detection-as-code, mapping-to-attack, reducing-false-positives, and red-team purple-teaming skills
- MITRE ATT&CK
