---
format: "v2"
name: "risk-assessment"
title: "Risk Assessment"
title_fr: "Évaluation des risques"
description: "Use when you need to identify, score, and prioritise security risks in a way people actually act on — turning \"this feels dangerous\" into a ranked, defensible register."
description_fr: "À utiliser pour identifier, noter et prioriser les risques de sécurité d'une façon sur laquelle les gens agissent réellement — transformer un \"ça semble dangereux\" en un registre classé et défendable."
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

Risk assessment is how security priorities get decided and defended to the people who fund them. Done well, it turns scattered worries into a ranked list tied to business impact; done badly, it's a colour-coded spreadsheet nobody reads. This skill covers producing an assessment that drives real decisions and underpins every framework in this domain.

### When to use it

Standing up a security programme, preparing for an audit or certification (ISO 27001, SOC 2, and friends all require it), evaluating a new system or vendor, or periodically re-baselining. It's the foundation the rest of GRC builds on — controls exist to treat risks, so you have to name the risks first.

### Procedure

1. **Scope it.** Decide what you're assessing — an application, a business unit, the whole organisation — and the boundaries. An unscoped assessment sprawls and finishes nothing.
2. **Identify assets and what threatens them.** What are you protecting (data, systems, processes), and what could go wrong (threats) exploiting what (vulnerabilities)? Keep each risk a concrete statement: "ransomware encrypts unbacked-up production data", not "cyber attack".
3. **Assess each risk on two axes — likelihood and impact.** Use a consistent scale (e.g. 1–5 each) and, critically, define what each level *means* up front so different people score the same risk the same way. Impact should map to business terms: money, downtime, data loss, regulatory penalty, reputation.
4. **Score and rank.** Likelihood × impact gives an inherent risk level. Rank the register by it so the top of the list is genuinely what matters most — this ordering is the entire point.
5. **Account for existing controls.** Distinguish **inherent** risk (before controls) from **residual** risk (after the controls you already have). Decisions are made on residual risk — that's the real exposure.
6. **Decide treatment for each.** Four options: **mitigate** (add/strengthen controls), **transfer** (insurance, outsourcing), **avoid** (stop doing the risky thing), or **accept** (with a named owner and review date). Every risk gets an explicit decision, not a default of silent acceptance.
7. **Record it in a risk register** with owner, treatment, and review date, and **revisit on a cadence** — risk isn't static, and an assessment done once and filed is worthless within months.

### Cheatsheet

```
risk statement:  <threat> exploits <vulnerability> causing <impact> to <asset>

score:  risk = likelihood (1-5)  ×  impact (1-5)
  define each level's meaning BEFORE scoring, so it's repeatable
  impact in business terms: $, downtime, data loss, fines, reputation

inherent risk  = before controls
residual risk  = after existing controls   <- decisions made here

treatment (pick one per risk):
  Mitigate  add/strengthen controls
  Transfer  insurance / outsource
  Avoid     stop the risky activity
  Accept    formal, with owner + review date

register row: risk | likelihood | impact | score | existing controls |
              residual | treatment | owner | review date
```

### Reading the assessment

- **A register genuinely ranked by residual risk** is the win — the top rows should be where effort and budget go, and leadership can see why.
- **Impact scored in vague terms** ("high") is weaker than impact in business terms ("~€500k + 2 days downtime") — the latter is what gets a risk funded. Push scoring toward concrete impact.
- **Everything scored "high"** means the scale isn't calibrated — if nothing is low, the assessment can't prioritise, which defeats its purpose.
- **Accepted risks with no owner or review date** aren't accepted, they're ignored — and after an incident, indistinguishable from negligence. Every acceptance needs a name and a date.
- **A register that never changes** between reviews is a sign it's being filed, not used.

### Making it drive decisions (the practice)

- **Calibrate the scale** with defined level meanings so scoring is consistent across assessors and over time.
- **Tie impact to the business**, not to technical severity alone — that's what turns a risk into a funded decision.
- **Make treatment explicit and owned.** A risk without a named owner and a decision drifts. The register is a commitment log, not a catalogue.
- **Feed it from real data**: vulnerability findings (cvss-in-context), threat intel, and incidents all sharpen likelihood and impact estimates.
- **Review on a cadence** and after significant change (new system, breach, major project) so the register stays current.

### Pitfalls

- **A spreadsheet nobody acts on.** If the assessment doesn't change what gets funded or fixed, it's compliance theatre. Rank by residual risk and route the top items to owners.
- **Uncalibrated scoring.** Without defined level meanings, scores are gut feelings dressed as numbers and aren't comparable.
- **Impact in technical, not business, terms.** "CVSS 9" doesn't move a budget; "€500k and a reportable breach" does.
- **Silent acceptance.** Un-owned, un-reviewed accepted risks are how known problems become incidents nobody decided to allow.
- **One-and-done.** Risk shifts constantly; a stale register misleads. Revisit it.

### References

- ISO/IEC 27005 (information security risk management)
- NIST SP 800-30 (Guide for Conducting Risk Assessments)
- ISO 31000 (risk management principles)
- FAIR (quantitative risk analysis) for maturing beyond qualitative scales

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.