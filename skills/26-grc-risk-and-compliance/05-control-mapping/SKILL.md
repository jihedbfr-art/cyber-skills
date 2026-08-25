---
format: "v2"
name: "control-mapping"
title: "Control Mapping"
title_fr: "Cartographie des contrôles"
description: "Use when an organisation faces multiple frameworks — mapping one control set to many frameworks so you implement a control once and satisfy several requirements instead of duplicating work."
description_fr: "À utiliser quand une organisation fait face à plusieurs référentiels — cartographier un ensemble de contrôles vers plusieurs référentiels pour implémenter un contrôle une fois et satisfaire plusieurs exigences au lieu de dupliquer le travail."
domain: "26-grc-risk-and-compliance"
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

Most organisations face multiple security frameworks and regulations at once — ISO 27001, SOC 2, PCI-DSS, NIS2, and more — and the requirements overlap enormously. Implementing and evidencing each separately duplicates huge amounts of work. Control mapping is building one unified control set and mapping it to all the frameworks, so a single control (and its evidence) satisfies many requirements. This skill covers control mapping, the efficiency practice that makes multi-framework compliance manageable.

### When to use it

When the organisation must comply with more than one framework/regulation (which is most of them beyond the smallest). It's the practice that turns "we have five separate compliance programmes" into "we have one control set mapped to five frameworks", saving enormous duplicated effort.

### Procedure

1. **Recognise the overlap — the core insight.** The major frameworks ask for largely the same things in different words: access control, encryption, incident response, risk management, logging, vendor management. ISO 27001's access-control requirement, SOC 2's, and PCI-DSS's are the same underlying control. Implementing it once and mapping it to all three is the efficiency win; treating them as separate triples the work.
2. **Build a unified control set — your controls, mapped outward.** Rather than organising by framework, define your organisation's actual controls once (a control catalogue), then map each control to the framework requirements it satisfies. This "implement once, map to many" structure is the foundation. Your control is the source of truth; the frameworks are views onto it.
3. **Map controls to framework requirements.** For each control, record which requirements (ISO Annex A clauses, SOC 2 TSC, PCI-DSS requirements, NIS2 duties) it satisfies. A crosswalk/matrix maps controls to all applicable frameworks, showing coverage and gaps per framework.
4. **Use published crosswalks as a starting point.** Many framework-to-framework mappings exist (the Secure Controls Framework, CIS mappings, NIST crosswalks); use them to accelerate the mapping rather than building from scratch, then tailor to your controls.
5. **Map evidence once, too.** The efficiency extends to evidence: a single piece of evidence (an access-review log, a policy) can satisfy the same control across multiple frameworks. Collect and organise evidence by control, so it serves all the framework audits it maps to (the evidence-and-audit-prep skill).
6. **Identify per-framework gaps.** The mapping shows where a framework requires something your control set doesn't cover; those gaps are the real work per framework, on top of the shared base. Most requirements are shared; the deltas are what's framework-specific.
7. **Maintain the mapping.** Frameworks update and new ones get added; a maintained control-to-framework mapping means adding a new framework is mostly mapping existing controls, not building anew. This is the long-term payoff.

### Cheatsheet

```
multiple frameworks (ISO/SOC2/PCI/NIS2...) overlap ENORMOUSLY — same things, different words
  implement + evidence each separately = massive DUPLICATED work
  control mapping = ONE control set mapped to MANY frameworks -> implement once, satisfy many

do
  1. RECOGNISE overlap (access control / encryption / IR / risk / logging / vendor = same underlying control)
  2. build a UNIFIED CONTROL SET (your controls once = source of truth ; frameworks = views onto it)
       organise by YOUR controls, map OUTWARD (not by framework)
  3. MAP controls -> framework requirements (crosswalk/matrix: ISO clauses, SOC2 TSC, PCI reqs, NIS2 duties)
       -> coverage + gaps per framework
  4. use PUBLISHED CROSSWALKS to start (Secure Controls Framework, CIS/NIST mappings) — don't build from scratch
  5. MAP EVIDENCE once too (one access-review log satisfies the control across frameworks) [evidence skill]
  6. per-framework GAPS = the real delta work (most reqs shared ; deltas = framework-specific)
  7. MAINTAIN -> adding a new framework = mostly mapping existing controls (the long-term payoff)
```

### Reading the mapping

- **Separate compliance programmes per framework** = massive duplicated effort; the same access control implemented and evidenced three times for three frameworks. A unified control set mapped outward is the efficiency win — this duplication is the problem control mapping solves.
- **A control catalogue as the source of truth, mapped to frameworks** = the right structure; your controls are real and permanent, the frameworks are views. Organising by framework (rather than by control) recreates the duplication.
- **Evidence collected by control, serving multiple framework audits** = the efficiency extended; one access-review log satisfies the requirement across ISO, SOC 2, and PCI. Collecting evidence per-framework duplicates it.
- **The mapping showing per-framework gaps** = the real per-framework work; most requirements are shared, and the deltas (what one framework needs that your controls don't cover) are the framework-specific effort.
- **Using published crosswalks** = accelerates the mapping; building framework-to-framework mappings from scratch wastes effort when crosswalks exist.
- **A maintained unified control set mapped to all frameworks with shared evidence** = multi-framework compliance made manageable; adding a framework becomes mostly mapping.

### Pitfalls

- **Running separate programmes per framework.** The core inefficiency — the frameworks overlap enormously, so implementing and evidencing each separately triples the work. Build one control set mapped to many.
- **Organising by framework instead of by control.** It recreates the duplication; your controls are the source of truth, frameworks are views onto them. Map outward from your controls.
- **Collecting evidence per-framework.** One piece of evidence satisfies the same control across frameworks; organise evidence by control to serve all the audits it maps to.
- **Building crosswalks from scratch.** Published mappings (SCF, CIS, NIST) accelerate it; use them and tailor.
- **Not maintaining the mapping.** Frameworks update and new ones appear; a stale mapping loses the "add a framework = mostly mapping" payoff.
- **Missing the per-framework deltas.** Most requirements are shared, but each framework has specifics your control set may not cover; the gaps are the real per-framework work.

### References

- The Secure Controls Framework (SCF), CIS Controls mappings, and NIST crosswalks
- The iso-27001-isms, soc-2-readiness, evidence-and-audit-prep, and gap-analysis skills
- GRC platforms that manage control-to-framework mappings
- NIST SP 800-53 / CSF (widely-mapped control catalogues)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.