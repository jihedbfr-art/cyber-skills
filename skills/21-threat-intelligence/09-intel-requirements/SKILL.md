---
format: "v2"
name: "intel-requirements"
title: "Intel Requirements"
title_fr: "Besoins en renseignement"
description: "Use when defining what intelligence to collect — establishing intelligence requirements so the programme answers real questions instead of collecting everything and hoping."
description_fr: "À utiliser pour définir quel renseignement collecter — établir des besoins de renseignement précis pour que le programme réponde à de vraies questions au lieu de tout collecter en espérant que ça serve."
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

A threat-intel programme without defined requirements collects everything and produces a firehose that answers no one's actual questions. Intelligence requirements flip this — you start from the questions the organisation needs answered ("who targets us and how", "are we exposed to X threat", "should we invest in Y defence") and collect *against* those. This skill covers defining and using intelligence requirements, the discipline that turns intel from undirected collection into a programme that delivers answers stakeholders actually need.

### When to use it

When establishing or refocusing a threat-intel programme, and whenever the intel being produced isn't landing (stakeholders aren't using it — often because it wasn't driven by their needs). It's the planning discipline that should drive collection, and it connects to tactical-vs-strategic (requirements exist at each level).

### Procedure

1. **Start from stakeholder decisions, not available data.** Intelligence requirements are the questions your consumers need answered to make decisions: the SOC ("what should we detect/block"), IR ("what campaigns and actors are active"), leadership ("what's our risk and where do we invest"). Requirements are consumer-driven — what they need to decide, not what intel happens to be available.
2. **Define Priority Intelligence Requirements (PIRs).** The key requirements the programme prioritises — typically framed as questions: "Which threat actors are targeting our sector and what are their TTPs?", "Are our credentials/data exposed?", "What are the emerging threats to our technology stack?". PIRs focus collection on what matters.
3. **Ground requirements in your threat model and business.** What's relevant depends on your sector, geography, technology, and crown jewels. A hospital, a bank, and a SaaS company have different PIRs. Requirements grounded in your actual context produce relevant intel; generic ones produce noise.
4. **Drive collection from the requirements.** Collection (feeds, sources, monitoring) should be chosen to *answer the PIRs*, not to gather everything. This focuses effort and cost on the intel that answers real questions and avoids drowning in irrelevant data.
5. **Map requirements to the intelligence levels.** Different requirements sit at tactical, operational, or strategic levels (that skill) with different audiences; a PIR about exposed credentials is tactical/operational, one about sector threat trends is strategic. Match collection and production accordingly.
6. **Review and refine requirements.** Business priorities, the threat landscape, and stakeholder needs change; requirements set once go stale. Revisit PIRs periodically and as circumstances change (a new business line, a new threat) so the programme stays aligned with real needs.
7. **Measure against requirements.** The programme's value is how well it answers its PIRs — not how much intel it collected. Assess whether stakeholders are getting the answers they need, and adjust collection where a requirement isn't being met.

### Cheatsheet

```
no requirements = collect everything -> firehose that answers no one's questions
  requirements flip it: start from the QUESTIONS stakeholders need answered, collect AGAINST them

start from STAKEHOLDER DECISIONS (not available data)
  SOC: what to detect/block | IR: active campaigns/actors | leadership: risk + investment

PIRs (Priority Intelligence Requirements) — framed as questions
  "which actors target our sector + their TTPs?" | "are our creds/data exposed?"
  | "emerging threats to our tech stack?"

GROUND in your threat model + business (sector/geo/tech/crown-jewels)
  hospital vs bank vs SaaS = different PIRs ; generic requirements = noise

DRIVE COLLECTION from requirements (answer the PIRs, not gather everything)
  -> focuses effort + cost, avoids drowning in irrelevant data

map requirements to LEVELS (tactical/operational/strategic) + audiences
REVIEW/refine (business + threat landscape change ; stale requirements = misaligned)
MEASURE against PIRs (answering the questions > volume collected)
```

### Reading the programme

- **Collection with no defined requirements** = a firehose producing intel nobody asked for; the programme is busy but not useful. Define PIRs and collect against them — this is the difference between directed intelligence and undirected data-gathering.
- **PIRs grounded in your sector/business** = relevant intel; requirements that reflect your actual threat model and crown jewels produce answers that matter, while generic requirements produce noise.
- **Collection driven by requirements** = focused effort and cost on what answers real questions; collecting everything wastes resources and buries the relevant intel.
- **Intel that stakeholders don't use** = often a requirements failure — it wasn't driven by what they need to decide. Reorient production around stakeholder decisions.
- **Stale requirements** (set once, never revisited) = a programme aligned to yesterday's needs; business and threats change, so PIRs must be refreshed.
- **A programme measured by how well it answers its PIRs** = the right metric; volume collected is a vanity measure, answering the questions is the value.

### Pitfalls

- **Collecting without requirements.** Gathering everything produces a firehose that answers no one's real questions; it's busy-work, not intelligence. Define PIRs first and collect against them.
- **Generic requirements.** Requirements not grounded in your sector, business, and threat model produce irrelevant noise; ground them in your actual context.
- **Data-driven instead of decision-driven requirements.** Starting from "what intel is available" rather than "what do stakeholders need to decide" produces intel that doesn't land. Start from the decisions.
- **Static requirements.** Business and threats evolve; PIRs set once go stale and misalign the programme. Review and refine.
- **Measuring by volume.** How much intel you collected says nothing about whether you answered the PIRs; measure against the requirements, not the firehose.

### References

- The tactical-vs-strategic and reporting-and-dissemination skills
- The threat-informed-detection skill (requirements drive detection priorities too)
- Standard CTI requirements frameworks (PIRs / intelligence requirements)
- The GRC risk-assessment skill (business context grounding)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.