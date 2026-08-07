---
name: mapping-intel-to-detection
domain: 21-threat-intelligence
description: Use when turning threat intelligence into detections and hunts — closing the loop so intel drives defence instead of sitting in a platform as unused reports.
difficulty: intermediate
tags: [threat-intel, detection, operationalising, hunting, integration]
tools: []
---

## Purpose

The most common failure of a threat-intel programme is that the intel doesn't reach defence — it accumulates in a platform as reports and indicators nobody operationalises. Intelligence only has value when it drives action: indicators become blocks and alerts, TTPs become detections and hunts. This skill covers closing that loop — turning threat intel into concrete detection and defensive action — the connective tissue between the intel domain and the detection/hunting domains.

## When to use it

Continuously, as the output side of the intel programme. It's what makes intelligence worth collecting; without it, all the collection, vetting, and enrichment produces nothing. It connects directly to threat-informed detection (which consumes intel to prioritise) and operationalising-a-hunt (the mirror discipline for hunting).

## Procedure

1. **Route indicators into automated detection and blocking.** Vetted indicators (past the false-positive checks) should flow automatically into the SIEM/EDR/firewall as detections and blocks — an IP/domain/hash tied to a threat becomes an alert or block without manual work. This is the tactical loop, and automation is essential (perishable indicators must land fast).
2. **Turn TTPs into behavioural detections — the higher-value loop.** Actor and campaign intelligence describes *how* adversaries operate; translate those TTPs into behavioural detections (Sigma rules mapped to ATT&CK — the detection domain). This is far more durable than indicator blocking, because it catches the technique regardless of infrastructure (Pyramid of Pain).
3. **Prioritise detection engineering with intel.** Intel about which actors and techniques target your sector should drive *what detections you build next* (threat-informed detection). Intel isn't just indicators to block — it's a prioritisation signal for the whole detection programme.
4. **Turn intel into hunts.** Intelligence about an actor's tradecraft or a campaign's techniques becomes hunt hypotheses (the hunting domain) — "actor X uses technique Y; let's hunt for Y in our environment". Intel gives hunts their best, most relevant hypotheses.
5. **Feed intel into enrichment.** Threat intel enriches alerts (is this alert's IP known-bad?) — the SOC alert-enrichment and enrichment-and-context skills consume intel to make alerts actionable. This is intel improving triage, not just creating new detections.
6. **Close the loop back — internal findings become intel.** Detections and hunts that find threats produce new intel (IoCs, TTPs) that feeds back into the intel programme — the most relevant intel of all is what actually hit you. The loop runs both ways.
7. **Measure that intel drives action.** Track detections created from intel, blocks applied, hunts run, and threats caught — evidence the intel is operationalised, not shelved. Intel that produces no detections or blocks isn't earning its cost.

## Cheatsheet

```
#1 intel-programme failure: intel sits in the platform, UNUSED. value = DRIVING ACTION.

close the loop
  INDICATORS -> automated detection + blocking (SIEM/EDR/firewall)
     tactical loop ; automate (perishable -> land fast) ; vetted past FP checks
  TTPs -> behavioural detections (Sigma + ATT&CK)   <- higher-value, durable
     catches the technique regardless of infrastructure (Pyramid of Pain)
  intel -> PRIORITISE detection engineering (threat-informed detection: what to build next)
  intel -> HUNTS (actor uses Y -> hunt for Y) — intel gives hunts their best hypotheses
  intel -> ENRICHMENT (is this alert's IP known-bad?) — improves triage, not just new rules

loop runs BOTH ways: internal detections/hunts -> new IoCs/TTPs -> back into intel
  (what actually hit you = the most relevant intel)

MEASURE: detections created / blocks applied / hunts run / threats caught
  no detections or blocks from intel = shelved, not earning its cost
```

## Reading the loop

- **Intel accumulating in a platform with no detections or blocks produced** = the classic, most-common failure; all the collection and enrichment produces nothing if it doesn't reach defence. Operationalising is what makes intel worth having.
- **Indicators flowing automatically into detection/blocking** = the tactical loop working; perishable indicators land fast without manual work. Manual indicator handling is too slow and doesn't scale.
- **TTPs turned into behavioural detections** = the higher-value loop; because it catches the technique regardless of infrastructure, it's far more durable than blocking indicators that rotate. This is where intel pays off most.
- **Intel prioritising the detection backlog** = intel as a strategic signal, not just indicators; what targets your sector should drive what you build (threat-informed detection).
- **Intel becoming hunt hypotheses** = intel giving hunts their most relevant targets; an actor's known TTPs are exactly what to hunt for.
- **Internal findings feeding back into intel** = the loop running both ways; what hit you is the best intel, and closing this loop makes the programme compound.
- **Measured detections/blocks/hunts/catches from intel** = proof it's operationalised; without these, the intel is shelved.

## Pitfalls

- **Intel that never reaches defence.** The dominant failure — collection, vetting, and enrichment produce nothing if intel sits unused in a platform. Operationalising into detection/blocking/hunting is the whole point.
- **Only blocking indicators.** Indicator blocking is the perishable, low-durability loop; the high-value move is turning TTPs into behavioural detections that survive infrastructure changes. Do both, but invest in TTPs.
- **Manual indicator handling.** Perishable indicators must land in tools fast and automatically; manual routing is too slow and doesn't scale.
- **Not using intel to prioritise detection.** Intel is a strategic signal for what to build, not just a stream of indicators to block; skipping this wastes its prioritisation value.
- **A one-way loop.** Not feeding internal detections/hunts back into intel misses the most relevant intelligence — what actually targeted you. Close the loop both ways.
- **Not measuring.** Without tracking detections/blocks/hunts produced from intel, you can't tell if it's operationalised or shelved.

## References

- The detection threat-informed-detection, writing-sigma-rules, and mapping-to-attack skills
- The threat-hunting operationalising-a-hunt and hunting-with-attack skills
- The SOC enrichment-and-context and detection alert-enrichment skills
- The Pyramid of Pain (indicators vs TTPs durability)
