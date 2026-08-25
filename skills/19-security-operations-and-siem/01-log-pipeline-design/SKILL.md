---
format: "v2"
name: "log-pipeline-design"
title: "Log Pipeline Design"
title_fr: "Conception du pipeline de logs"
description: "Use when designing the pipeline that gets logs into a SIEM — collection, parsing, and normalisation so the right data arrives usable, because everything downstream depends on it."
description_fr: "À utiliser pour concevoir le pipeline qui achemine les logs vers un SIEM — collecte, parsing et normalisation pour que les bonnes données arrivent exploitables, car tout le reste du SOC en dépend."
domain: "19-security-operations-and-siem"
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

A SIEM is only as good as the data flowing into it, and getting that data in — collected from the right sources, parsed correctly, normalised to a common shape — is where a lot of SOCs quietly fail. Detections that assume a field exists, hunts that can't correlate across sources, and analysts fighting inconsistent data all trace back to the pipeline. This skill covers designing a log pipeline that delivers the right telemetry in a usable form, the foundation everything else in the SOC stands on.

### When to use it

Building or overhauling a SIEM deployment, onboarding new log sources, or diagnosing why detections and hunts are unreliable (often a pipeline problem, not a rule problem). It pairs with the detection log-source-coverage skill — that decides *what* to collect, this handles *getting it in usable*.

### Procedure

1. **Collect the right sources — driven by detection and visibility needs.** Don't collect everything or whatever's easy; collect the telemetry your detections and hunts need (from log-source-coverage): endpoint, authentication, network (DNS/proxy/firewall), cloud audit, and key application logs. Every source has a purpose or it's cost without value.
2. **Parse into structured fields.** Raw log lines are unusable for correlation; parse each source into structured fields (a timestamp, source/dest, user, action) so detections and searches can reference them reliably. Broken or missing parsing is why a field a rule needs isn't there.
3. **Normalise to a common schema — the high-value step.** Different sources call the same thing different names (`src_ip`, `source.ip`, `ClientIP`). Normalise to a common schema (e.g. the Elastic Common Schema, or your SIEM's data model) so a detection or hunt can query across sources uniformly. Without normalisation, correlation is a per-source nightmare.
4. **Enrich at ingest where it helps** — add context (asset, geo, initial threat-intel tags) in the pipeline so it's available to every detection (ties into enrichment skills), rather than looked up repeatedly later.
5. **Handle time correctly.** Normalise timestamps to UTC and preserve the event's true time (not just ingest time); time is the backbone of correlation and timelines, and skew/timezone errors corrupt everything (the forensics timeline skill's warning applies).
6. **Design for reliability and volume.** The pipeline must handle the log volume without dropping events (buffering, backpressure), and you should monitor it — a silently-failed log source is a blind spot nobody notices until an incident. Alert on sources that stop sending.
7. **Balance completeness against cost** (the retention/cost skill) — route high-value security logs to the SIEM and lower-value/high-volume data to cheaper storage, rather than paying premium ingest for everything.

### Cheatsheet

```
SIEM is only as good as the pipeline feeding it. stages:

1. COLLECT (right sources, by detection/visibility need — not "everything")
     endpoint | auth | network (DNS/proxy/fw) | cloud audit | key apps
2. PARSE into structured fields (raw lines can't be correlated)
     broken parsing = the field your rule needs isn't there
3. NORMALISE to a common schema (ECS / SIEM data model)  <- high-value
     src_ip / source.ip / ClientIP -> one field ; else cross-source query = nightmare
4. ENRICH at ingest (asset/geo/intel tags) -> available to every detection
5. TIME: normalise to UTC, keep true event time (not ingest time) — backbone of correlation
6. RELIABILITY: handle volume (buffer/backpressure), MONITOR sources
     silently-dead source = unnoticed blind spot -> ALERT when a source stops
7. COST: high-value -> SIEM ; high-volume/low-value -> cheaper storage
```

### Reading the pipeline

- **Detections failing because a field is missing/inconsistent** = a parsing or normalisation problem, not a rule bug; the pipeline isn't delivering the data in the shape the rule expects. This is a common root cause mistaken for a detection issue.
- **Hunts that can't correlate across sources** = missing normalisation; the same entity has different field names per source, so cross-source queries break. A common schema fixes it.
- **A log source that silently stopped sending** = an unnoticed blind spot; detections on it quietly stop working and nobody knows until an incident. Monitor sources and alert on gaps.
- **Timestamp chaos** (ingest time used as event time, mixed timezones) = broken correlation and timelines; time handling is foundational and easy to get wrong.
- **Paying premium SIEM ingest for high-volume low-value logs** = cost without security value; route by value (retention/cost skill).
- **Right sources, parsed, normalised, time-correct, monitored, cost-tiered** = a pipeline the rest of the SOC can rely on.

### Pitfalls

- **Collecting everything (or whatever's easy).** Ingesting without purpose wastes cost and buries signal; collect by detection and visibility need. Conversely, missing a needed source is a blind spot.
- **Skipping normalisation.** The single most impactful pipeline gap — without a common schema, every cross-source detection and hunt fights inconsistent field names. Normalise.
- **Unmonitored sources.** A log source that dies silently removes coverage nobody notices; alert when expected data stops arriving.
- **Mishandling time.** Using ingest time as event time or mixing timezones corrupts correlation and timelines. Normalise to UTC, preserve true event time.
- **Treating pipeline problems as detection problems.** Chasing rule logic when the real issue is parsing/normalisation wastes effort; check the data shape first.

### References

- Elastic Common Schema (ECS) and SIEM data-model documentation
- Log-shipping/pipeline tools: Vector, Logstash, Fluentd, and SIEM-native collectors
- The detection log-source-coverage, enrichment-and-context, and log-retention-and-cost skills
- NIST SP 800-92 (log management)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.