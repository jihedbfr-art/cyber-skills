# 18 — Detection Engineering

Treating detections like code: written to a hypothesis, tested against real telemetry, versioned, and tuned. A rule that fires on everything is as useless as one that fires on nothing. This domain is about the middle.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [writing-sigma-rules](01-writing-sigma-rules/SKILL.md) | Portable detection logic in Sigma | ✅ |
| 02 | [detection-as-code](02-detection-as-code/SKILL.md) | Version, test, and deploy rules through CI | ✅ |
| 03 | [mapping-to-attack](03-mapping-to-attack/SKILL.md) | Tie every rule to an ATT&CK technique | ✅ |
| 04 | [reducing-false-positives](04-reducing-false-positives/SKILL.md) | Tune without creating blind spots | ✅ |
| 05 | [edr-detection-logic](05-edr-detection-logic/SKILL.md) | Endpoint behavioural rules | ✅ |
| 06 | [log-source-coverage](06-log-source-coverage/SKILL.md) | Know what you can and can't see | ✅ |
| 07 | [testing-detections](07-testing-detections/SKILL.md) | Validate with atomic tests and emulation | ✅ |
| 08 | [alert-enrichment](08-alert-enrichment/SKILL.md) | Give the analyst context, not just a hit | ✅ |
| 09 | [detection-metrics](09-detection-metrics/SKILL.md) | Measure coverage and quality | ✅ |
| 10 | [threat-informed-detection](10-threat-informed-detection/SKILL.md) | Build from real intel, not guesses | ✅ |

This domain is complete (10/10). Start at `writing-sigma-rules` and pair it with `mapping-to-attack`; `log-source-coverage` is the foundation underneath all of it.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>