---
format: "v2"
name: "enrichment-pipelines"
title: "Enrichment Pipelines"
title_fr: "Pipelines d'enrichissement"
description: "Use when building automated enrichment for indicators — adding context (reputation, WHOIS, relationships, geolocation) to raw IoCs so they become actionable intelligence."
description_fr: "À utiliser pour construire l'enrichissement automatisé des indicateurs — ajouter du contexte (réputation, WHOIS, relations, géolocalisation) aux IoC bruts pour en faire du renseignement exploitable."
domain: "21-threat-intelligence"
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

A raw indicator — an IP, a domain, a hash — is data, not intelligence. What makes it actionable is context: is it known-bad, how old is the domain, what else is it linked to, where is it hosted, what threat is it associated with. Enrichment pipelines add that context automatically, at scale, so indicators arrive as intelligence rather than as bare values an analyst has to research one by one. This skill covers building automated enrichment that turns the IoC firehose into contextualised, prioritisable intel.

### When to use it

When you're handling more indicators than you can research manually (always, at any scale), and when downstream consumers (detection, blocking, analysts) need context to act. It sits between collection/vetting and consumption, and connects to the SOC's alert-enrichment (this enriches the intel; that enriches the alerts).

### Procedure

1. **Identify the enrichment that makes indicators actionable.** For each indicator type, what context informs a decision? Common enrichments:
   - **Reputation** — is it flagged malicious by reputation services / other feeds?
   - **Domain/IP metadata** — WHOIS (registration date, registrar — new domains are suspicious), geolocation, hosting provider, passive DNS (what else resolved here).
   - **Relationships** — what other indicators, malware, or actors is it associated with (from your platform's links).
   - **File context** — for hashes, sandbox/VT results, malware family, prevalence.
2. **Automate it in a pipeline.** Enrichment must be automatic — an indicator entering the platform triggers lookups that decorate it with context (MISP modules, Cortex analysers, or a custom pipeline). Manual per-indicator research doesn't scale past a handful.
3. **Enrich with false-positive-avoidance in mind.** Enrichment is also how you catch the false-positive risks from the vetting skill — a WHOIS/reputation lookup revealing an IP is a major CDN or a domain is a legitimate service flags "don't block this". Enrichment protects against acting on bad indicators.
4. **Add confidence and prioritisation.** Enrichment data should feed a confidence/priority score — a newly-registered domain flagged by multiple sources and linked to a known actor is high-priority; an old IP with one weak flag is low. This scoring is what makes the enriched intel prioritisable.
5. **Keep enrichment current.** Reputation and context change; a domain benign yesterday may be flagged today (and vice versa). Re-enrich as needed rather than treating a one-time lookup as permanent truth.
6. **Manage API costs and rate limits.** Enrichment services have quotas and costs; enrich efficiently (cache results, prioritise which indicators to enrich deeply) rather than hammering every indicator against every service.
7. **Feed enriched intel to consumers.** The contextualised indicators flow into detection (with confidence), blocking (past false-positive checks), and analysts (with the research pre-done). Enrichment's value is realised downstream.

### Cheatsheet

```
raw indicator = DATA. context = what makes it INTELLIGENCE (actionable).
  enrichment pipeline adds context automatically, at scale.

enrichment types
  REPUTATION       flagged malicious by services/feeds?
  DOMAIN/IP META   WHOIS (registration DATE — new = suspicious), geo, hosting, passive DNS
  RELATIONSHIPS    linked indicators / malware / actors (platform links)
  FILE             hash -> sandbox/VT, malware family, prevalence

AUTOMATE (MISP modules / Cortex analysers / custom) — manual doesn't scale
enrich for FALSE-POSITIVE avoidance too: WHOIS/rep reveals CDN / legit service -> "don't block"
add CONFIDENCE/PRIORITY score (new domain + multi-source + known actor = high)
keep CURRENT (reputation changes ; re-enrich, not one-time truth)
manage API cost/rate limits (cache, prioritise deep enrichment)
feed CONSUMERS: detection (w/ confidence) + blocking (past FP checks) + analysts (research pre-done)
```

### Reading enrichment

- **A raw indicator with no context** = data an analyst must research one by one; enrichment automates that research so indicators arrive actionable. The difference between data and intelligence.
- **Enrichment revealing an indicator is a CDN/legitimate service** = a false-positive catch; the WHOIS/reputation lookup that flags "don't block this" prevents an outage. Enrichment is a key false-positive defence, not just added detail.
- **A newly-registered domain flagged by multiple sources and linked to a known actor** = the enrichment stacking into a high-priority indicator; the combined context is what enables prioritisation. Bare indicators can't be prioritised this way.
- **Stale enrichment** (a one-time lookup treated as permanent) = reputation changes; an indicator's context can shift. Re-enrich rather than trusting old lookups.
- **Manual enrichment** = doesn't scale past a handful of indicators; the pipeline must be automated or it becomes the bottleneck.
- **Enriched, scored, current indicators flowing to detection, blocking, and analysts** = the pipeline delivering intelligence, not data.

### Pitfalls

- **Not automating.** Manual enrichment doesn't scale; past a few indicators it's the bottleneck. Build a pipeline (MISP modules, Cortex, custom).
- **Skipping false-positive-avoidance enrichment.** Enrichment that reveals CDNs and legitimate services is a key defence against acting on bad indicators; without it you block things you shouldn't.
- **One-time enrichment.** Reputation and context change; treating a single lookup as permanent truth leads to stale, wrong decisions. Re-enrich.
- **Ignoring API costs/limits.** Hammering every indicator against every service burns quota and money; cache and prioritise which indicators get deep enrichment.
- **Enriching without scoring.** Context that doesn't feed a confidence/priority signal leaves indicators un-prioritisable; the point is to make them actionable and rankable.
- **Enrichment that doesn't reach consumers.** Context is only valuable downstream (detection, blocking, analysts); wire it through.

### References

- MISP enrichment modules and Cortex analysers
- The ioc-collection-and-vetting, mapping-intel-to-detection skills and SOC enrichment-and-context skill
- Passive DNS, WHOIS, and reputation service documentation
- STIX (structured context representation)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.