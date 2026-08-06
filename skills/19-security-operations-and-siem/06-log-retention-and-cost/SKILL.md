---
name: log-retention-and-cost
domain: 19-security-operations-and-siem
description: Use when balancing log retention against SIEM cost — keeping the data you need for detection, hunting, and compliance without paying premium ingest for everything.
difficulty: intermediate
tags: [soc, siem, retention, cost, logging]
tools: []
---

## Purpose

SIEMs typically charge by data volume, so "log everything forever" is a fast route to a budget blowout — but under-retaining leaves you unable to detect, hunt, or investigate. Log retention and cost is the balancing act: keep the right data, for the right time, in the right tier of storage. This skill covers making that trade-off deliberately, so security value drives spend rather than the SIEM bill dictating (and quietly gutting) your visibility.

## When to use it

Designing or optimising a SIEM deployment, when SIEM costs are escalating, or when retention limits are hurting investigations (you needed logs from 90 days ago and they're gone). It's a recurring tension in every SOC and a place where cost pressure can silently erode security if not managed deliberately.

## Procedure

1. **Understand the drivers of retention need.** Three things dictate how long to keep what: **detection** (recent data for real-time rules), **hunting/investigation** (weeks to months, because intrusions are often found late — the average dwell time means you need history), and **compliance** (regulations may mandate specific retention periods). Each source's retention should be driven by which of these apply to it.
2. **Tier storage by value and access need — the key lever.** Not all logs need to be in the expensive, hot, searchable SIEM tier. Route high-value, frequently-queried security logs to the SIEM (hot); send high-volume or lower-value data to cheaper cold/archive storage that's still retrievable when needed. This is how you keep long retention without paying premium ingest for everything.
3. **Filter and reduce at ingest.** Drop or aggregate low-value, high-volume noise before it hits the priced SIEM tier — verbose debug logs, redundant events, health checks. Reducing volume at the source cuts cost without losing security-relevant data (done carefully — don't drop what a detection needs).
4. **Match retention to dwell time, not just compliance.** A common mistake is retaining only for the compliance minimum, then finding an intrusion that started before your window. Because attackers often dwell for weeks or months undetected, retention for *investigation* usually needs to exceed the bare compliance minimum. Set it by when you'd realistically need the data.
5. **Keep critical logs longer and protected.** Authentication, audit, and high-value security logs warrant longer retention and tamper protection (the auditd/cloudtrail off-host logging discipline) — these are exactly what you need in an incident.
6. **Monitor and forecast cost.** Track ingest volume and cost trends; a new noisy source can spike the bill. Forecasting prevents surprise overruns and the panic response of slashing retention (which cuts visibility).
7. **Review the balance periodically** — sources, volumes, and needs change; revisit what's retained where.

## Cheatsheet

```
SIEM charges by VOLUME -> "log everything forever" = budget blowout
under-retain -> can't detect/hunt/investigate. balance deliberately.

retention drivers (per source)
  DETECTION      recent data for real-time rules (short)
  HUNT/INVESTIGATE  weeks-months — intrusions found LATE (dwell time) -> need history
  COMPLIANCE     regulated minimums (may be specific)

KEY LEVER: TIER storage by value + access need
  hot/SIEM (expensive, searchable)  <- high-value, frequently-queried security logs
  cold/archive (cheap, retrievable) <- high-volume / lower-value data
  -> long retention WITHOUT premium ingest for everything

reduce at ingest: drop/aggregate low-value noise (debug/health/redundant) — carefully
match retention to DWELL TIME, not just compliance minimum
  (attackers dwell weeks/months -> intrusion may predate a short window)
critical logs (auth/audit): longer + tamper-protected
monitor + FORECAST cost (noisy new source = bill spike) ; review periodically
```

## Reading the trade-off

- **Retaining only the compliance minimum** = a frequent trap; an intrusion that started before your window can't be investigated because the logs are gone. Match retention to realistic dwell time, which usually exceeds the compliance floor.
- **Everything in the hot SIEM tier** = paying premium ingest for high-volume low-value data; tiering high-volume logs to cheap archive keeps them retrievable at a fraction of the cost. The main cost lever.
- **Cost pressure slashing retention across the board** = silently gutting hunting and investigation capability; the response to a bill spike should be tiering and filtering, not blindly cutting the history you need for incidents.
- **A noisy new source spiking the bill** = why forecasting matters; catch it before it forces a panic cut. Reduce that source at ingest.
- **Critical auth/audit logs under-retained or unprotected** = exactly the data you need in an incident, missing or tamperable. These warrant longer, protected retention.
- **Value-tiered storage, dwell-time-matched retention, ingest filtering, protected critical logs, forecasted cost** = the balance — full visibility at sustainable cost.

## Pitfalls

- **Retaining to the compliance minimum only.** Intrusions are found late; if your window is shorter than the dwell time, the investigation hits a wall. Retain for investigation, not just compliance.
- **Everything in the expensive tier.** Paying hot-tier ingest for high-volume low-value logs blows the budget; tier by value and access need.
- **Slashing retention under cost pressure.** The knee-jerk fix guts visibility; tier and filter instead of cutting the history you need.
- **No ingest filtering.** Sending verbose noise to the priced SIEM wastes money; drop/aggregate low-value data at the source (without dropping what detections need).
- **Under-retaining or exposing critical logs.** Auth/audit logs are incident essentials; keep them longer and tamper-protected.
- **No cost forecasting.** A noisy source spikes the bill and forces a panic response; monitor and forecast volume.

## References

- The log-pipeline-design, detection log-source-coverage, and auditd/cloudtrail logging skills
- NIST SP 800-92 (log management, including retention)
- Compliance frameworks with logging requirements (PCI-DSS, HIPAA, etc.) via the GRC domain
- SIEM vendor pricing/tiering documentation
