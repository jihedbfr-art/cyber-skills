# 20 — Threat Hunting

Assume the alerts missed something and go looking. Hunting starts from a hypothesis — "if an attacker did X, I'd see Y in this log" — and searches the telemetry to prove or kill it. Findings that repeat become detections and feed domain 18.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [hypothesis-driven-hunting](01-hypothesis-driven-hunting/SKILL.md) | Frame a hunt you can actually answer | ✅ |
| 02 | hunting-with-attack | Use ATT&CK to pick what to hunt | TODO |
| 03 | beaconing-detection | Find C2 in the noise of outbound traffic | TODO |
| 04 | lateral-movement-hunting | Spot movement across hosts | TODO |
| 05 | living-off-the-land | Catch abuse of legitimate binaries | TODO |
| 06 | dns-and-proxy-hunting | Mine web and DNS logs for badness | TODO |
| 07 | anomaly-baselining | Know normal so you can see abnormal | TODO |
| 08 | endpoint-hunting | Process, persistence, and injection hunts | TODO |
| 09 | data-stacking | Frequency analysis to surface outliers | TODO |
| 10 | operationalising-a-hunt | Turn a finding into a lasting detection | TODO |

Begin with `hypothesis-driven-hunting` (done) — the method matters more than any single query.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>