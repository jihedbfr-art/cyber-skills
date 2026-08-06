---
name: metrics-and-mttr
domain: 19-security-operations-and-siem
description: Use when measuring SOC performance honestly — the metrics that show whether the SOC is effective and where it's struggling, avoiding the vanity numbers that look good and mean nothing.
difficulty: intermediate
tags: [soc, metrics, mttr, performance, measurement]
tools: []
---

## Purpose

SOC metrics are easy to game and easy to misread — "we closed 10,000 alerts this month" says nothing about whether the SOC caught the attacks that mattered. Measuring a SOC honestly means tracking speed, effectiveness, and health in ways that reveal real performance and point at what to fix. This skill covers the metrics that matter — chiefly the mean-time-to-X measures — and how to read them without being fooled by vanity numbers or perverse incentives.

## When to use it

Assessing and improving SOC performance, and reporting it to leadership. Good metrics drive the right improvements (faster detection, better triage, less noise) and justify resourcing; bad metrics drive gaming (closing alerts fast regardless of quality) and misplaced confidence.

## The metrics that matter

- **MTTD (mean time to detect)** — how long from an attack starting to it being detected. The headline effectiveness measure; long MTTD means attackers operate undetected for too long.
- **MTTR (mean time to respond/resolve)** — how long from detection to containment/resolution. Measures response speed once something's caught.
- **MTTT / time to triage** — how long an alert waits before an analyst works it. Reveals queue backlog and staffing issues.
- **Alert volume and false-positive rate** — the workload and its quality; high volume with high false positives means the SOC is drowning in noise (ties into detection metrics).
- **Escalation accuracy** — are alerts escalated correctly (true incidents escalated, false positives not)? Measures triage quality.
- **Coverage** — what the SOC can actually detect (from the detection metrics/mapping skills).

## Procedure

1. **Anchor on the mean-time-to-X metrics.** MTTD and MTTR are the core because they measure what the SOC exists to do — catch and respond to attacks quickly. Track them, trend them, and drive them down. A SOC that detects and responds fast is effective regardless of how many alerts it closes.
2. **Avoid vanity and perverse-incentive metrics.** "Alerts closed" and "tickets handled" are the trap — they reward volume and speed of *closing*, which incentivises closing alerts without proper investigation (including real ones as false positives). Never make closure rate the goal; it corrupts triage quality.
3. **Measure quality alongside speed.** Fast is only good if it's also right — pair MTTR with escalation accuracy and false-negative signals (attacks missed, found later). A SOC that's fast but misses real attacks isn't performing, however good its speed looks.
4. **Measure health, not just output.** Alert volume vs analyst capacity, false-positive rate, and analyst burnout indicators show whether the SOC is sustainable. A SOC hitting its speed targets while burning out is heading for failure.
5. **Use metrics diagnostically.** Each metric should point at a fix: high MTTD → detection/telemetry gaps; high time-to-triage → staffing or noise; low escalation accuracy → triage training or runbooks; high false-positives → detection tuning. Metrics that don't drive action are overhead.
6. **Report honestly to leadership** in terms of effectiveness and gaps (MTTD/MTTR trends, coverage, capacity) — not vanity counts — to justify investment and show trajectory. Beware presenting closure rates as success.
7. **Beware gaming.** Any metric becomes a target people optimise for; watch for optimisation that games the number without improving security (closing alerts fast, tuning out inconvenient detections).

## Cheatsheet

```
"we closed 10,000 alerts" = says NOTHING about catching what mattered.

core metrics (the mean-time-to-X)
  MTTD  attack start -> detected      headline effectiveness (long = attacker free too long)
  MTTR  detected -> contained/resolved  response speed
  time to TRIAGE  alert waits before worked   queue/staffing signal

quality + health (fast only matters if RIGHT)
  escalation accuracy (TP escalated, FP not)   triage quality
  false-negative signals (attacks missed, found later)
  alert volume vs capacity + FP rate + burnout   sustainability

AVOID vanity / perverse incentives
  "alerts closed" / "tickets handled" -> rewards closing fast -> real alerts
     closed as false positives. NEVER make closure rate the goal.

use DIAGNOSTICALLY (each -> a fix)
  high MTTD -> detection/telemetry gap | high triage time -> staffing/noise
  low escalation accuracy -> training/runbooks | high FP -> tune detections

report EFFECTIVENESS + gaps to leadership (not counts) ; beware gaming (any metric = target)
```

## Reading the metrics

- **Closure rate / alerts-handled presented as success** = the vanity trap; it rewards fast closing regardless of correctness, which incentivises closing real alerts as false positives to hit the number. Never optimise for it. This is the most damaging SOC metric mistake.
- **High MTTD** = attackers operate undetected too long; the SOC's core effectiveness measure is failing, pointing at detection coverage or telemetry gaps. The metric that matters most.
- **Fast MTTR but attacks found late by others** = speed without effectiveness; the SOC resolves what it catches fast but misses real attacks. Pair speed with quality/false-negative signals.
- **High time-to-triage** = alerts queuing before anyone works them; a staffing or noise problem — analysts can't keep up. Points at capacity or false-positive reduction.
- **Good speed metrics with analyst burnout** = unsustainable; the SOC is hitting targets by overworking people and heading for failure. Measure health.
- **A metric being gamed** (detections tuned out to improve numbers, alerts closed without investigation) = the metric became a target; watch for optimisation that improves the number, not security.
- **MTTD/MTTR trended, paired with quality and health, driving fixes** = honest, useful SOC measurement.

## Pitfalls

- **Vanity/closure metrics.** "Alerts closed" rewards volume and speed of closing, incentivising closing real alerts as false positives. The single most corrosive SOC metric — never make it the goal.
- **Speed without quality.** Fast MTTR means nothing if the SOC misses real attacks; pair speed metrics with escalation accuracy and false-negative signals.
- **Ignoring SOC health.** Hitting speed targets while burning out analysts or drowning in noise is unsustainable; measure capacity and false-positive load.
- **Metrics that don't drive action.** A dashboard nobody acts on is overhead; each metric should point at a specific improvement.
- **Reporting counts to leadership.** It looks like progress but misleads; report MTTD/MTTR trends, coverage, and gaps.
- **Ignoring gaming.** Any metric becomes a target; watch for optimisation that games the number (tuning out detections, closing without investigating) rather than improving security.

## References

- The detection detection-metrics, alert-triage-workflow, and on-call-and-escalation skills
- The vuln-mgmt reporting-to-stakeholders skill (same anti-vanity-metric discipline)
- SANS SOC metrics and MITRE detection maturity resources
- Goodhart's Law (any metric that becomes a target ceases to be a good measure)
