---
format: "v2"
name: "iso-27001-isms"
title: "Iso 27001 Isms"
title_fr: "SMSI ISO 27001"
description: "Use when standing up or running an ISO 27001 information security management system — building the risk-driven management framework, not just a binder of policies, and getting certified."
description_fr: "À utiliser pour mettre en place ou faire vivre un système de management de la sécurité de l'information ISO 27001 — construire le cadre de gestion piloté par le risque, pas juste un classeur de politiques, et obtenir la certification."
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

ISO 27001 is the international standard for an Information Security Management System (ISMS) — a systematic, risk-driven way of managing security. The common misunderstanding is that it's a checklist of controls to implement; it's actually a *management system* with the risk assessment at its core, and the controls follow from the risk. This skill covers standing up and running an ISO 27001 ISMS effectively, so certification reflects real security management rather than paperwork.

### When to use it

Building a formal security programme, pursuing ISO 27001 certification (often required by customers/contracts), or improving an existing ISMS. It's the framework that structures a security programme, and understanding it as risk-driven (not control-driven) is what makes it valuable rather than bureaucratic.

### Procedure

1. **Understand it's a management system, risk-driven — the key insight.** ISO 27001 requires a *system* for managing security: defined scope, leadership commitment, a risk assessment and treatment process, controls chosen to address the identified risks, and continual improvement (the Plan-Do-Check-Act cycle). The **risk assessment drives which controls you implement** — you don't implement all controls, you implement the ones your risks require. Missing this leads to a checkbox exercise.
2. **Define the scope.** What parts of the organisation, systems, and information the ISMS covers. Scope shapes everything and the certification; too broad is unmanageable, too narrow is meaningless to customers.
3. **Do the risk assessment (the core).** Identify risks to your information (the risk-assessment skill), assess them, and decide treatment. This is the engine of the ISMS — the whole point is managing risk systematically, and the controls are selected to treat the assessed risks.
4. **Select controls and produce the Statement of Applicability (SoA).** ISO 27001's Annex A lists reference controls (the 2022 version has 93, in four themes); the SoA documents which you apply, which you don't, and *why* — justified by your risk assessment. The SoA is a central certification artifact and must trace to the risks.
5. **Implement the ISMS processes**, not just the controls — the management elements: leadership/governance, competence and awareness, documented information, internal audit, management review, incident management, and continual improvement. These *processes* are what distinguish a management system from a set of controls.
6. **Run internal audits and management reviews.** The ISMS must audit itself and be reviewed by leadership regularly — evidence that it's a living system, not a one-time setup. These are certification requirements and the mechanism of continual improvement.
7. **Get certified and maintain it.** Certification is a two-stage external audit (documentation review, then implementation audit), followed by ongoing surveillance audits. Certification isn't the end — the ISMS must keep running (risk reassessment, audits, improvement) or it lapses.

### Cheatsheet

```
ISO 27001 = a MANAGEMENT SYSTEM for security (not a control checklist)
  KEY INSIGHT: RISK-DRIVEN — the risk assessment drives WHICH controls you implement
  (missing this = checkbox exercise)

build the ISMS
  1. it's a SYSTEM: scope + leadership + risk process + controls-from-risk + continual improvement (PDCA)
  2. SCOPE (shapes everything + certification ; too broad unmanageable / too narrow meaningless)
  3. RISK ASSESSMENT (the core/engine): identify -> assess -> treat (controls treat the risks)
  4. controls + STATEMENT OF APPLICABILITY (SoA): Annex A reference controls (2022: 93, 4 themes)
       -> which apply / don't + WHY (justified by risk) — central cert artifact, traces to risks
  5. implement ISMS PROCESSES not just controls: governance, awareness, documented info,
       internal audit, management review, incident mgmt, continual improvement
       (the processes = what makes it a management system)
  6. INTERNAL AUDITS + MANAGEMENT REVIEWS (living system, not one-time ; cert requirements)
  7. CERTIFY (2-stage external audit) + MAINTAIN (surveillance audits ; keeps running or lapses)
```

### Reading the ISMS

- **An ISMS treated as a control checklist** = the fundamental misunderstanding; ISO 27001 is risk-driven, and controls should be selected to treat assessed risks, not implemented as a blanket list. A checkbox ISMS is bureaucracy without security value.
- **A Statement of Applicability that traces to the risk assessment** = the correct linkage; the SoA justifies control choices by risk. An SoA that just marks all controls "applicable" without risk justification misses the point.
- **The management processes present** (governance, internal audit, management review, continual improvement) = what makes it a *management system*; an ISMS with controls but no processes is just a control set, and won't certify.
- **Internal audits and management reviews happening regularly** = a living ISMS; their absence means it's a one-time setup that will lapse and fail surveillance audits.
- **Scope well-defined** (meaningful to customers, manageable in size) = the ISMS covers what matters; a too-narrow scope is meaningless certification, too-broad is unmanageable.
- **A risk-driven, process-complete, audited, maintained ISMS** = real security management that certification reflects — not paperwork.

### Pitfalls

- **Treating it as a control checklist.** The core misunderstanding — ISO 27001 is a risk-driven management system; controls follow from the risk assessment. A checkbox approach produces bureaucracy without security value.
- **An SoA not justified by risk.** Marking controls applicable without linking to the risk assessment misses the standard's logic and weakens certification. The SoA must trace to risks.
- **Implementing controls but not the management processes.** Governance, internal audit, management review, and continual improvement are what make it a management *system*; without them it's just a control set and won't certify.
- **A one-time setup.** The ISMS must keep running (risk reassessment, audits, improvement); certification lapses if it becomes static. Maintain it.
- **Bad scoping.** Too broad is unmanageable; too narrow is meaningless to customers. Scope deliberately.

### References

- ISO/IEC 27001:2022 (the standard) and ISO/IEC 27002 (control guidance)
- The risk-assessment, control-mapping, policy-writing, and evidence-and-audit-prep skills
- Certification body guidance (two-stage audit, surveillance)
- ISO 27005 (information security risk management — the risk engine)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.