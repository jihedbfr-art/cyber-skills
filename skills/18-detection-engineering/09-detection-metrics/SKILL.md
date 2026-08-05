---
name: detection-metrics
domain: 18-detection-engineering
description: Use when measuring the quality of a detection programme — the metrics that show whether detections work and where to improve, instead of vanity counts of how many rules exist.
difficulty: intermediate
tags: [detection, metrics, quality, coverage, measurement]
tools: []
---

## Purpose

"How many detection rules do you have?" is a vanity metric — it says nothing about whether you'd catch an attack. Measuring a detection programme well means tracking quality and coverage: what you can detect, how reliably, how fast, and how much noise it costs. This skill covers the metrics that actually reveal a programme's health and drive improvement, so effort goes where it matters rather than toward inflating a rule count.

## When to use it

Assessing and improving a detection programme, and reporting its state to leadership. Good metrics turn a subjective "we have lots of rules" into an evidence-based view of "here's what we catch, here's the gaps, here's the noise" — which drives both engineering priorities and investment.

## The metrics that matter

- **Coverage (quality-weighted)** — what fraction of relevant ATT&CK techniques you detect, weighted by detection quality, not just presence (from the mapping-to-attack skill). This answers "what can we catch?".
- **Detection efficacy** — do detections actually fire on the techniques they target (from the testing-detections skill)? Validated detections vs assumed ones. This answers "do they work?".
- **False-positive rate / precision** — how much of what a detection fires on is real. High-noise detections cost the SOC and erode trust (the reducing-false-positives skill). This answers "can we trust them?".
- **Mean time to detect (MTTD)** — how fast a real attack is caught after it starts. This answers "are we fast enough?".
- **Alert volume and analyst load** — how many alerts the programme generates and whether the SOC can keep up. Detections that overwhelm the SOC aren't helping.

## Procedure

1. **Avoid the vanity metric.** "Number of rules" is the trap — it rewards adding rules regardless of quality, and a thousand noisy or untested rules are worse than fifty good ones. Don't lead with it, and don't optimise for it.
2. **Measure coverage against ATT&CK, quality-weighted.** Track which relevant techniques you detect and how well, so the metric reflects real capability, not a rule tally. Trend it over time to show progress.
3. **Measure efficacy through testing.** Track the fraction of detections validated (they fire on their technique) vs untested/assumed. A programme where most detections are unvalidated is riskier than its rule count suggests.
4. **Measure noise and precision.** Track false-positive rates per detection and overall; identify the noisiest rules (usually a few generate most of the fatigue) as tuning priorities. Precision protects analyst trust.
5. **Measure speed and load.** MTTD shows whether detections catch attacks fast enough to matter; alert volume vs SOC capacity shows whether the programme is sustainable.
6. **Use metrics to drive action, not just report.** Each metric should point somewhere: low coverage in a relevant area → build detections there; low efficacy → test and fix; high false-positives → tune; high MTTD → improve the detections or telemetry. Metrics that don't change behaviour are overhead.
7. **Report the honest picture to leadership** in terms of capability and gaps (not rule counts), to justify investment and show trajectory (ties into the reporting discipline).

## Cheatsheet

```
VANITY (don't lead with / optimise for): "number of rules"
  1000 noisy/untested rules < 50 good ones

metrics that MATTER (each -> an action)
  COVERAGE (quality-weighted)   % relevant ATT&CK detected, weighted by quality
                                 -> gap in relevant area? build there
  EFFICACY                       % detections VALIDATED (fire on their technique)
                                 -> mostly untested? test + fix
  FALSE-POSITIVE / precision      how much of what fires is real
                                 -> noisiest rules = tuning priority
  MTTD                            time to detect a real attack after it starts
                                 -> too slow? improve detections/telemetry
  ALERT VOLUME vs SOC capacity    is it sustainable? overwhelming SOC = not helping

drive ACTION, not just reporting ; report CAPABILITY + gaps to leadership (not counts)
```

## Reading the metrics

- **A high rule count presented as success** = the vanity trap; it says nothing about whether you'd catch an attack, and often hides noisy, untested rules. Reframe around coverage, efficacy, and precision.
- **Low quality-weighted coverage in a relevant area** = a real capability gap; a technique attackers targeting you use, that you can't catch well. This drives where to build next.
- **Mostly unvalidated detections** = your real coverage is lower than the map suggests; efficacy metrics expose the gap between assumed and proven. Prioritise testing.
- **A few detections generating most false positives** = the tuning priorities; fixing them reclaims analyst trust and time. Precision metrics surface them.
- **High MTTD** = attacks run for too long before detection; the issue may be the detections or the telemetry/speed. It points at whether the programme is fast enough to matter.
- **Alert volume exceeding SOC capacity** = the programme is unsustainable; more detections here make it worse, not better. Load metrics reveal it.
- **Coverage, efficacy, precision, MTTD, and load trended and driving action** = a measured, improving programme.

## Pitfalls

- **Optimising for rule count.** The classic vanity metric — it rewards quantity over quality and can make the programme *worse* (more noise, more untested rules). Measure capability, not tally.
- **Coverage without quality-weighting.** Counting techniques with "a rule" overstates capability; weight by whether the detection actually detects well.
- **Ignoring efficacy.** Coverage and rule counts assume detections work; without testing metrics you don't know how many actually fire. Measure validated vs assumed.
- **Metrics that don't drive action.** A dashboard nobody acts on is overhead; each metric should point to a specific improvement.
- **Reporting counts to leadership.** It misleads (looks like progress) and doesn't justify investment well; report capability and gaps instead.

## References

- The mapping-to-attack, testing-detections, and reducing-false-positives skills
- The vuln-mgmt reporting-to-stakeholders skill (same anti-vanity-metric discipline)
- MITRE ATT&CK (coverage baseline) and SANS detection-engineering metrics resources
