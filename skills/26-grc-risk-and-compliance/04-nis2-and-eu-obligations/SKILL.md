---
format: "v2"
name: "nis2-and-eu-obligations"
title: "Nis2 And Eu Obligations"
title_fr: "NIS2 et obligations européennes"
description: "Use when mapping EU cybersecurity regulatory duties to controls — NIS2, GDPR security obligations, and DORA — so the organisation meets its legal requirements, not just best practice."
description_fr: "À utiliser pour cartographier les obligations réglementaires européennes en cybersécurité vers des contrôles — NIS2, obligations de sécurité du RGPD et DORA — pour que l'organisation respecte ses obligations légales, pas seulement les bonnes pratiques."
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

Beyond voluntary frameworks (ISO, SOC 2), organisations operating in the EU face *legal* cybersecurity obligations — NIS2, GDPR's security requirements, and DORA for financial entities — with real penalties for non-compliance. These are mandatory, not best-practice, and they carry specific duties (risk management, incident reporting timelines, governance accountability). This skill covers understanding the main EU cybersecurity obligations and mapping them to controls, so the organisation meets its legal requirements. (It's a starting map, not legal advice — regulatory specifics need qualified counsel.)

### When to use it

When the organisation operates in or serves the EU and needs to understand and meet its regulatory cybersecurity duties. It's distinct from voluntary frameworks because non-compliance carries legal penalties, making it a priority for affected organisations. Given the legal stakes, involve legal/compliance counsel — this skill orients you, it doesn't replace advice.

### The main EU obligations

- **NIS2 (Directive on network and information security)** — expands the earlier NIS directive to more sectors ("essential" and "important" entities). Requires risk-management measures, incident reporting (with tight timelines — an early warning within 24 hours, notification within 72 hours), supply-chain security, and — notably — **management accountability** (leadership can be held personally responsible for compliance). Applies to a broad range of sectors.
- **GDPR (security obligations)** — beyond privacy, GDPR requires "appropriate technical and organisational measures" to protect personal data (Article 32), and mandates breach notification (to the authority within 72 hours, and to affected individuals for high-risk breaches). The security and breach-notification duties are the cybersecurity-relevant parts (the IR communication skill).
- **DORA (Digital Operational Resilience Act)** — for the EU financial sector; specific requirements for ICT risk management, incident reporting, resilience testing, and third-party (ICT provider) risk. Prescriptive for financial entities.

### Procedure

1. **Determine which obligations apply — the first step.** Do you fall under NIS2 (which sector, essential vs important)? Do you process EU personal data (GDPR)? Are you a financial entity (DORA)? Applicability depends on sector, size, and what data/services you handle. Get this right (with counsel) — it defines your legal duties.
2. **Map the obligations to specific duties.** Each regulation imposes concrete requirements — risk management measures, incident-reporting timelines, governance/accountability, supply-chain/third-party controls, testing. List the specific duties that apply to you.
3. **Map the duties to your controls.** Much of what these regulations require, you may already do (risk management, incident response, access control) — the task is mapping your existing controls to the regulatory duties and identifying gaps (the control-mapping and gap-analysis skills). Frameworks like ISO 27001 cover much of the technical ground.
4. **Address the reporting-timeline duties specifically.** The incident-reporting deadlines (NIS2's 24h/72h, GDPR's 72h) are strict and easy to miss under incident pressure; your incident-response process must include the regulatory notification steps and timelines (the IR communication skill), or you breach the law even while handling the incident well technically.
5. **Address governance and accountability.** NIS2's management accountability means leadership must be engaged and can be personally liable; this is a governance requirement, not just a technical one. Ensure the board/leadership understand and own their obligations.
6. **Address supply-chain/third-party duties.** NIS2 and DORA both require managing third-party/ICT-provider risk (the third-party-risk-management skill); the regulations make vendor risk a legal obligation, not just good practice.
7. **Work with legal/compliance and document.** Regulatory compliance needs qualified legal input on applicability and interpretation, and documented evidence of the measures taken. This skill orients the security side; counsel handles the legal specifics.

### Cheatsheet

```
EU = LEGAL cybersecurity obligations (mandatory, penalties) beyond voluntary ISO/SOC 2
  (orienting map, NOT legal advice — involve counsel)

main obligations
  NIS2   essential/important entities (broad sectors) ; risk-mgmt measures + INCIDENT REPORTING
         (early warning 24h / notification 72h) + supply-chain security
         + MANAGEMENT ACCOUNTABILITY (leadership personally liable) 
  GDPR   security = Art.32 "appropriate technical + organisational measures" + BREACH NOTIFICATION
         (authority 72h ; individuals if high-risk)
  DORA   EU FINANCIAL sector ; ICT risk mgmt + incident reporting + resilience TESTING + third-party ICT risk

do
  1. APPLICABILITY (with counsel): NIS2 sector? GDPR (EU personal data)? DORA (financial)? — defines duties
  2. map obligations -> specific DUTIES (risk mgmt, reporting timelines, governance, supply-chain, testing)
  3. map duties -> your CONTROLS (much already done — ISO 27001 covers a lot) + gaps
  4. REPORTING TIMELINES (24h/72h strict) -> bake into IR process (or breach the law while handling well)
  5. GOVERNANCE/ACCOUNTABILITY (NIS2 leadership liability — board owns it)
  6. SUPPLY-CHAIN/third-party (NIS2 + DORA make vendor risk a LEGAL duty)
  7. work with legal/compliance + DOCUMENT the measures
```

### Reading the obligations

- **Uncertainty about which regulations apply** = the first thing to resolve (with counsel); applicability (NIS2 sector, GDPR data, DORA financial) defines your legal duties, and getting it wrong means either non-compliance or wasted effort. Determine scope first.
- **Incident-reporting timelines not built into the IR process** = a legal-breach risk even when you handle an incident well technically; NIS2's 24h/72h and GDPR's 72h are strict and easy to miss under pressure. The regulatory notification steps must be in the IR runbook.
- **Leadership unaware of NIS2 accountability** = a governance gap; NIS2 can hold management personally liable, so the board must understand and own the obligations. This is a distinctive NIS2 requirement.
- **Third-party risk treated as optional** = NIS2 and DORA make it a legal duty; vendor/ICT-provider risk management is required, not just good practice (the third-party-risk-management skill).
- **Existing controls not mapped to regulatory duties** = you may already meet much of the requirement but can't demonstrate it; map controls to duties and identify the real gaps (control-mapping/gap-analysis).
- **Applicability determined, duties mapped to controls, reporting timelines in the IR process, governance owned, documented with counsel** = regulatory obligations met, not just best practice.

### Pitfalls

- **Treating EU obligations as best practice.** They're legal requirements with penalties; non-compliance has consequences voluntary frameworks don't. Prioritise accordingly.
- **Getting applicability wrong.** Whether NIS2/GDPR/DORA apply defines your duties; determine it with counsel first, or you under- or over-comply.
- **Missing reporting timelines.** The 24h/72h deadlines are strict and easy to miss during an incident; bake the regulatory notification steps into the IR process, or breach the law while handling the incident well.
- **Ignoring governance/accountability.** NIS2's management liability is distinctive; leadership must be engaged and own the obligations, not delegate them away.
- **Overlooking third-party duties.** NIS2 and DORA make vendor risk a legal obligation; it's not optional.
- **Skipping legal counsel.** Regulatory interpretation and applicability need qualified legal input; this skill orients the security side but doesn't replace advice.

### References

- NIS2 Directive (EU 2022/2555), GDPR (Art. 32, 33, 34), DORA (EU 2022/2554) — with legal counsel
- The control-mapping, gap-analysis, third-party-risk-management, and IR communication-during-incidents skills
- ENISA guidance on NIS2 implementation
- National transpositions of NIS2 (member-state specifics)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.