---
name: ioc-collection-and-vetting
domain: 21-threat-intelligence
description: Use when gathering indicators of compromise from feeds and reports — collecting them without drowning in junk, and vetting them so you don't act on false or stale data.
difficulty: beginner
tags: [threat-intel, iocs, feeds, vetting, quality]
tools: [misp]
---

## Purpose

Threat intelligence starts with indicators — the IPs, domains, hashes, and URLs associated with threats — but raw feeds are a firehose of variable quality, and acting on bad indicators causes false positives, wasted effort, and even outages (blocking a legitimate service). This skill covers collecting IoCs from feeds and reports and, crucially, vetting them so what reaches your detection and blocking is accurate, relevant, and current.

## When to use it

The intake step of a threat-intel programme, and whenever ingesting a new feed or report. It feeds everything downstream (enrichment, detection mapping, blocking), so the quality gate here determines the value of the whole pipeline — garbage in, garbage out.

## Procedure

1. **Collect from multiple, appropriate sources.** IoCs come from commercial feeds, open-source feeds (OSINT, abuse lists), ISAC/sector sharing, vendor reports, and your own incidents and hunts (the most relevant of all). Combine sources for coverage, but weight them by reliability.
2. **Assess source reliability.** Not all feeds are equal — some are curated and accurate, others are noisy or full of expired data. Track which sources produce actionable indicators vs false positives, and weight accordingly. A feed that repeatedly causes false positives is negative value.
3. **Vet indicators before acting — the key gate.** Raw IoCs must be checked before they drive blocking or alerting:
   - **False-positive risk:** is this IP a shared-hosting/CDN/major-service address, or a domain that's legitimate-but-abused (a compromised CDN, a common service)? Blocking these causes outages. This vetting is what prevents self-inflicted damage.
   - **Accuracy:** is the indicator actually associated with the threat, or a mis-attribution?
   - **Relevance:** does this threat apply to your environment/sector?
4. **Track indicator age and expiry.** IoCs decay — a malicious IP gets cleaned up, a domain gets sinkholed, infrastructure rotates. Stale indicators cause false positives (blocking a now-legitimate host) and waste. Age indicators and expire them.
5. **Add context and confidence.** An indicator without context (what threat, what confidence, what to do) gets misapplied. Attach the threat it relates to, a confidence level, and the recommended action.
6. **Prioritise by the Pyramid of Pain.** Weight indicators by durability (the pyramid-of-pain skill) — hashes/IPs are cheap and perishable, behavioural indicators are durable. Collect the low-tier for fast blocking but value the high-tier.
7. **Feed vetted indicators onward** — into a platform (MISP), enrichment, and detection, with their context and confidence intact.

## Cheatsheet

```
raw feeds = firehose of VARIABLE quality ; bad IoCs = false positives / outages
  garbage in -> garbage out. vetting is the value gate.

collect (multiple sources, weight by reliability)
  commercial | OSINT/abuse lists | ISAC/sector | vendor reports | YOUR incidents+hunts (most relevant)

assess SOURCE reliability (track actionable vs FP per source ; noisy feed = negative value)

VET before acting (the key gate)
  FALSE-POSITIVE risk: shared-hosting/CDN/major-service IP? legit-but-abused domain?
    -> blocking these = OUTAGE. this vetting prevents self-inflicted damage.
  ACCURACY: actually tied to the threat, or mis-attribution?
  RELEVANCE: applies to your environment/sector?

AGE + expiry: IoCs decay (IP cleaned, domain sinkholed, infra rotates) -> stale = FP
CONTEXT + confidence: what threat / how sure / what action (else misapplied)
prioritise by PYRAMID OF PAIN (hash/IP perishable ; behavioural durable)
```

## Reading the indicators

- **An indicator that's a shared-hosting IP, CDN, or legitimate-but-abused domain** = a false-positive/outage risk if blocked naively; vetting for this is the single most important check, because acting on it can take down access to a legitimate service. Never block without this vetting.
- **A high-reliability source's indicators** = more trustworthy and actionable; track which sources earn this and weight them. A feed that repeatedly false-positives is worse than no feed.
- **A stale indicator** (old, from a now-cleaned host) = a false-positive waiting to happen; aging and expiry prevent blocking infrastructure that's no longer malicious.
- **An indicator with no context or confidence** = it'll be misapplied; without knowing the threat and confidence, an analyst can't judge how to act. Attach both.
- **Indicators from your own incidents/hunts** = the most relevant of all; what actually hit you is higher-value than any generic feed. Prioritise internal sources.
- **Vetted, contexted, aged, durability-weighted indicators** = the quality input the rest of the pipeline needs.

## Pitfalls

- **Acting on unvetted indicators.** Blocking a shared IP, CDN, or legitimate abused domain from a raw feed causes outages; vetting for false-positive risk is what prevents self-inflicted damage. Never auto-block raw feed data.
- **Treating all sources equally.** Feeds vary wildly in quality; a noisy feed generates false positives and is negative value. Assess and weight source reliability.
- **Ignoring indicator decay.** IoCs go stale as infrastructure is cleaned/rotated; blocking on old indicators false-positives. Age and expire them.
- **No context or confidence.** Bare indicators get misapplied; attach the threat, confidence, and recommended action.
- **Valuing volume over quality.** A huge feed of low-quality indicators is worse than a small vetted set; garbage in, garbage out. Vet before you value.

## References

- MISP documentation (indicator management, feeds, warninglists for false-positive avoidance)
- The pyramid-of-pain, enrichment-pipelines, and mapping-intel-to-detection skills
- STIX/TAXII (structured indicator formats)
- The malware extracting-iocs and threat-hunting operationalising skills (internal IoC sources)
