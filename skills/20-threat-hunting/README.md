# 20 — Threat Hunting

Assume the alerts missed something and go looking. Hunting starts from a hypothesis — "if an attacker did X, I'd see Y in this log" — and searches the telemetry to prove or kill it. Findings that repeat become detections and feed domain 18.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [hypothesis-driven-hunting](01-hypothesis-driven-hunting/SKILL.md) | Frame a hunt you can actually answer | ✅ |
| 02 | [hunting-with-attack](02-hunting-with-attack/SKILL.md) | Use ATT&CK to pick what to hunt | ✅ |
| 03 | [beaconing-detection](03-beaconing-detection/SKILL.md) | Find C2 in the noise of outbound traffic | ✅ |
| 04 | [lateral-movement-hunting](04-lateral-movement-hunting/SKILL.md) | Spot movement across hosts | ✅ |
| 05 | [living-off-the-land](05-living-off-the-land/SKILL.md) | Catch abuse of legitimate binaries | ✅ |
| 06 | [dns-and-proxy-hunting](06-dns-and-proxy-hunting/SKILL.md) | Mine web and DNS logs for badness | ✅ |
| 07 | [anomaly-baselining](07-anomaly-baselining/SKILL.md) | Know normal so you can see abnormal | ✅ |
| 08 | [endpoint-hunting](08-endpoint-hunting/SKILL.md) | Process, persistence, and injection hunts | ✅ |
| 09 | [data-stacking](09-data-stacking/SKILL.md) | Frequency analysis to surface outliers | ✅ |
| 10 | [operationalising-a-hunt](10-operationalising-a-hunt/SKILL.md) | Turn a finding into a lasting detection | ✅ |

This domain is complete (10/10). Begin with `hypothesis-driven-hunting` — the method matters more than any single query; `operationalising-a-hunt` is what makes the programme compound.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>