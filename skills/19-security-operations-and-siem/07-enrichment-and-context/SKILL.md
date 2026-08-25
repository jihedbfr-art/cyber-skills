---
format: "v2"
name: "enrichment-and-context"
title: "Enrichment And Context"
title_fr: "Enrichissement et contexte"
description: "Use when wiring asset, identity, and threat context into the SOC's data so every alert and query carries the context analysts need — the operational backbone behind fast triage."
description_fr: "À utiliser pour connecter le contexte actifs, identités et menaces aux données du SOC afin que chaque alerte et requête porte le contexte dont les analystes ont besoin — la colonne vertébrale opérationnelle d'un triage rapide."
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

The detection domain's alert-enrichment skill covers enriching individual alerts; this one covers the SOC-wide plumbing that makes it possible — wiring asset, identity, and threat-intelligence data into the SIEM/pipeline so context is available everywhere, for every alert, query, and hunt. Without this backbone, enrichment is a manual scramble per alert; with it, context is just *there*. This skill covers building the SOC's context layer, the infrastructure that makes fast, informed triage the default.

### When to use it

Setting up or maturing a SOC's data platform, when analysts lack the context to triage efficiently, or when detections and hunts can't reference asset/identity information. It's the operational counterpart to alert-enrichment (which uses this context) and log-pipeline-design (which delivers the events this context decorates).

### The context sources

- **Asset context** — from the asset inventory (the vuln-mgmt skill): criticality, environment, owner, role. Answers "how important is this system?".
- **Identity context** — from the IAM/directory: user role, privilege, department, account type (service vs human). Answers "who is this and what can they do?".
- **Threat intelligence** — IoCs, reputation, actor associations (the threat-intel domain). Answers "is this known-bad?".
- **Network/business context** — network zones, business function, data sensitivity. Answers "where and what does this touch?".

### Procedure

1. **Integrate the context sources into the SOC platform.** Connect the asset inventory, identity directory, and threat-intel feeds to the SIEM/enrichment layer so their data is queryable and can decorate events. This integration is the backbone; enrichment of individual alerts depends on it existing.
2. **Enrich at the right point — ideally at ingest or alert creation.** Decorate events/alerts with context as they flow through the pipeline (log-pipeline-design) or via SOAR at alert creation, so the context is present without a per-alert manual lookup. Some context (fast-changing threat intel) may be looked up at query time instead.
3. **Keep the context data current — the make-or-break factor.** Stale context is worse than none: an asset inventory that's out of date mislabels criticality, a threat-intel feed that's not refreshed misses new indicators, an identity source that lags shows wrong privileges. The enrichment is only as good as the freshness of its sources. Automate updates.
4. **Make context queryable for hunting, not just alerting.** The same context that speeds triage supercharges hunting — being able to filter by asset criticality or user privilege in a hunt query is powerful. Build the context so it serves both.
5. **Prioritise asset and identity context** — these change triage decisions the most (a DC vs a test box, an admin vs an intern) and are the most commonly missing. Get these in first.
6. **Avoid context overload.** Enrich with what informs decisions, not everything available; drowning alerts in irrelevant context is as unhelpful as having none (the alert-enrichment discipline).
7. **Maintain the integrations** — sources, APIs, and schemas change; a broken context integration silently degrades every alert's usefulness.

### Cheatsheet

```
this = the SOC-wide plumbing that makes per-alert enrichment possible
  without it: manual context scramble per alert ; with it: context is just THERE

context sources
  ASSET     inventory: criticality/env/owner/role   ("how important?")
  IDENTITY  directory: role/privilege/dept/type      ("who + what can they do?")
  THREAT INTEL  IoCs/reputation/actor                 ("known-bad?")
  NETWORK/BUSINESS  zone/function/data-sensitivity    ("where + what touches?")

build
  integrate sources -> SIEM/enrichment layer (the backbone)
  enrich at INGEST / alert creation (present without manual lookup)
    (fast-changing intel: query-time lookup)
  KEEP CONTEXT CURRENT — stale context worse than none (automate updates)  <- make/break
  make context QUERYABLE for hunting too (not just alerting)
  prioritise ASSET + IDENTITY (change triage most, most often missing)
  avoid context OVERLOAD (inform decisions, not everything)
  maintain integrations (broken = silent degradation)
```

### Reading the context layer

- **Analysts manually looking up asset/identity/intel per alert** = the context backbone is missing or not wired into alerts; building the integration makes context automatic and triage fast. This is the whole point.
- **Stale context** (out-of-date inventory, unrefreshed intel, lagging identity) = worse than none, because it actively misleads — an asset labelled low-criticality that's actually a DC, or wrong privilege on an account. Freshness is make-or-break; automate updates.
- **Missing asset/identity context specifically** = the highest-impact gap, since these change triage decisions the most and are most often absent. Prioritise them.
- **Context available for alerting but not hunting** = a missed opportunity; the same context makes hunts far more powerful. Build it queryable for both.
- **Alerts drowning in irrelevant context** = overload; enrich with what informs the decision, not everything. Relevance over volume.
- **Integrated, fresh, prioritised, queryable context feeding every alert and hunt** = the backbone that makes fast, informed SOC work the default.

### Pitfalls

- **Stale context.** The biggest failure — outdated asset/identity/intel data actively misleads triage (wrong criticality, wrong privilege, missed indicators). Automate freshness; stale is worse than absent.
- **No context backbone.** Without integrated sources, enrichment is a manual per-alert scramble that doesn't scale. Build the integration once so context is everywhere.
- **Missing asset/identity context.** These matter most for triage and are commonly absent; prioritise wiring them in.
- **Context only for alerts.** Making it queryable for hunting too multiplies its value; don't limit it to alerting.
- **Context overload.** Too much irrelevant context buries the useful; enrich for decisions, not completeness.
- **Unmaintained integrations.** A broken source silently degrades every alert; monitor and maintain them.

### References

- The detection alert-enrichment, log-pipeline-design skills
- The vuln-mgmt asset-inventory, IAM, and threat-intelligence domains (context sources)
- SANS SOC data-platform and enrichment resources

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.