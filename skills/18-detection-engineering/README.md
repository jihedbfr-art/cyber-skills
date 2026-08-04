# 18 — Detection Engineering

Treating detections like code: written to a hypothesis, tested against real telemetry, versioned, and tuned. A rule that fires on everything is as useless as one that fires on nothing. This domain is about the middle.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [writing-sigma-rules](01-writing-sigma-rules/SKILL.md) | Portable detection logic in Sigma | ✅ |
| 02 | detection-as-code | Version, test, and deploy rules through CI | TODO |
| 03 | mapping-to-attack | Tie every rule to an ATT&CK technique | TODO |
| 04 | reducing-false-positives | Tune without creating blind spots | TODO |
| 05 | edr-detection-logic | Endpoint behavioural rules | TODO |
| 06 | log-source-coverage | Know what you can and can't see | TODO |
| 07 | testing-detections | Validate with atomic tests and emulation | TODO |
| 08 | alert-enrichment | Give the analyst context, not just a hit | TODO |
| 09 | detection-metrics | Measure coverage and quality | TODO |
| 10 | threat-informed-detection | Build from real intel, not guesses | TODO |

Start at `writing-sigma-rules` (done) and pair it with `mapping-to-attack`.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>