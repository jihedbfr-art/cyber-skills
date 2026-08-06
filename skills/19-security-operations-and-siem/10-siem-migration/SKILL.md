---
name: siem-migration
domain: 19-security-operations-and-siem
description: Use when migrating from one SIEM to another — moving detections, data sources, and operations to a new platform without going blind during the transition.
difficulty: advanced
tags: [soc, siem, migration, detections, transition]
tools: []
---

## Purpose

SIEM migrations happen — for cost, capability, or a platform going end-of-life — and they're risky, because the SIEM is the SOC's nervous system. Done badly, a migration creates a detection gap during the switchover where attacks go unseen, or loses detections and tuning built up over years. This skill covers migrating SIEMs without going blind: running both in parallel, porting detections deliberately, and validating the new platform before you cut over. It draws on the detection-as-code discipline that makes migration far less painful.

## When to use it

Planning or executing a SIEM platform change. It's infrequent but high-stakes, and the difference between a smooth migration and a dangerous gap is entirely in the planning — specifically, resisting the urge to cut over before the new platform is proven.

## Procedure

1. **Run both SIEMs in parallel — the core risk-mitigation.** Don't switch off the old SIEM until the new one is proven to detect what the old one did. Ingest into both during the transition so there's no window where neither is fully watching. This parallel period is what prevents the detection gap that makes migrations dangerous; it costs (double ingest for a while) but the cost is worth avoiding blindness.
2. **Inventory what you're migrating.** Detections/use cases (and their tuning), data sources, integrations, dashboards, and response runbooks. Years of accumulated detection logic and tuning are the real asset — losing them is the hidden cost of a rushed migration.
3. **Port detections deliberately, not blindly.** Detection logic must be translated to the new platform's query language and data model — and this is where portable, version-controlled detections (detection-as-code, Sigma) pay off enormously, since portable rules convert far more easily than platform-locked ones. Re-validate each ported detection fires on the new platform (testing-detections skill); a translated rule isn't proven until tested.
4. **Re-onboard and normalise data sources** on the new platform (log-pipeline-design), confirming each source is collected, parsed, and normalised correctly — a source that silently doesn't onboard is a blind spot on the new SIEM.
5. **Validate coverage before cutover.** Compare detection coverage on the new platform against the old (mapping/metrics skills) and confirm the new one detects at least what the old did — ideally test with atomic tests. Cutting over before this validation is the classic migration mistake.
6. **Migrate operations and train analysts.** The SOC has to work the new platform — runbooks, workflows, and analyst familiarity all need to transfer. A technically-migrated SIEM that analysts can't use effectively is a soft failure.
7. **Cut over gradually and keep a rollback path.** Move in stages, validate at each, and don't decommission the old SIEM until the new one is fully proven in production. Keep the ability to fall back if something's wrong.

## Cheatsheet

```
the SIEM is the SOC's nervous system -> migration risk = a DETECTION GAP or lost detections

CORE: run BOTH in parallel until the new one is PROVEN
  ingest into both during transition -> no window where neither is watching
  (double-ingest cost < the cost of going blind)

inventory to migrate: detections+TUNING | data sources | integrations | dashboards | runbooks
  (years of tuning = the real asset; rushed migration loses it)

port detections DELIBERATELY
  translate to new query language + data model
  detection-as-code / Sigma pays off HUGELY (portable > platform-locked)
  RE-VALIDATE each ported rule fires on the new platform (translated != proven)

re-onboard + normalise data sources (silent non-onboard = new-SIEM blind spot)
VALIDATE coverage vs old BEFORE cutover (mapping/metrics + atomic tests)
migrate operations + TRAIN analysts (unusable new SIEM = soft failure)
cutover GRADUALLY + keep ROLLBACK ; decommission old only when new is proven
```

## Reading the migration

- **Cutting over before validating the new platform detects what the old did** = the classic, dangerous mistake; it creates a detection gap where attacks go unseen. Run in parallel and validate coverage first — this is non-negotiable.
- **Detections lost or not ported** = years of accumulated tuning and coverage gone; the hidden cost of a rushed migration. Inventory and port deliberately, re-validating each.
- **Platform-locked detections** = painful to migrate (rewrite each in the new query language); this is exactly why detection-as-code and portable Sigma rules are worth adopting *before* you ever need to migrate.
- **A data source that silently didn't onboard** on the new SIEM = a blind spot nobody notices; confirm every source is collected, parsed, and normalised on the new platform.
- **A translated rule assumed to work** = unproven until tested; translation introduces errors (field names, syntax). Re-validate each ported detection fires.
- **Analysts unable to work the new platform** = a soft failure even if the tech migrated; train them and migrate the operational layer.
- **Parallel-run, deliberately-ported, coverage-validated, gradual cutover with rollback** = a safe migration with no blind window.

## Pitfalls

- **Cutting over too early.** Decommissioning the old SIEM before the new one is proven creates a detection gap — the core migration danger. Run both in parallel until validated.
- **Losing detections and tuning.** Years of accumulated logic and false-positive tuning are the real asset; a rushed migration drops them. Inventory and port deliberately.
- **Assuming translated rules work.** Porting to a new query language/data model introduces errors; a translated rule is unproven until re-tested on the new platform.
- **Silent data-source gaps.** A source that doesn't onboard correctly is a blind spot on the new SIEM; validate every source's collection, parsing, and normalisation.
- **Skipping coverage validation.** Cutting over without comparing new-vs-old coverage risks detecting less than before. Validate first.
- **Neglecting the human/operational migration.** A technically-migrated SIEM analysts can't use effectively fails softly; migrate runbooks and train.

## References

- The detection-as-code, testing-detections, mapping-to-attack skills (portability + validation)
- The log-pipeline-design and log-source-coverage skills (data source re-onboarding)
- Sigma (portable detections that ease migration)
- SANS SIEM deployment/migration resources
