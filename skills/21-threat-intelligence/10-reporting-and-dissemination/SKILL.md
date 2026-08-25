---
format: "v2"
name: "reporting-and-dissemination"
title: "Reporting And Dissemination"
title_fr: "Rapports et diffusion"
description: "Use when writing and delivering threat-intel reports — producing intelligence people actually read and act on, and getting it to the right consumers in the right form and time."
description_fr: "À utiliser pour rédiger et diffuser des rapports de renseignement sur la menace — produire du renseignement que les gens lisent et exploitent réellement, et le faire parvenir aux bons destinataires sous la bonne forme au bon moment."
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

Intelligence that isn't communicated well is wasted, no matter how good the analysis. Reporting and dissemination is the last mile — turning analysis into reports people actually read and act on, and getting them to the right consumers in the right form and at the right time. This skill covers producing usable intelligence reporting and delivering it effectively, the closing discipline that determines whether all the collection and analysis produces value.

### When to use it

The output side of every intel product, and whenever intel reports aren't landing (nobody reads them, or reads and doesn't act). It's the communication counterpart to intel-requirements — requirements define what to answer, this delivers the answers usably. It connects to tactical-vs-strategic (form per audience) and the vuln-mgmt reporting discipline.

### Procedure

1. **Write for the audience and their decision.** Match the report to the consumer (tactical/operational/strategic — that skill): the SOC needs concise, actionable indicators and detection content; leadership needs business-framed risk in plain language. The same analysis becomes different reports for different audiences. A report that doesn't fit its reader doesn't get used.
2. **Lead with the answer, not the analysis.** Open with the bottom line — what's the threat, how does it affect us, what should we do. Analysts and executives alike need the key judgement up front, with supporting analysis available below for those who want it (the BLUF — bottom line up front — discipline).
3. **Make it actionable.** Every report should lead to a decision or action: block these, build this detection, adjust this defence, make this investment. Intelligence that describes a threat without a "so what / now what" leaves the reader to figure out the response. State the recommended action.
4. **Convey confidence and sourcing appropriately.** Intelligence carries uncertainty; express confidence levels (high/medium/low) and the basis for judgements so consumers can weight them. Overstating certainty misleads decisions; hiding uncertainty behind vague language is unusable. Be honest about what you know and don't.
5. **Deliver in the right form and timing.** Tactical intel needs to be fast and machine-consumable (indicators into tools, not a PDF); strategic intel can be a periodic briefing. Perishable intel delivered slowly is worthless; match the delivery mechanism and cadence to the intel's shelf life and audience.
6. **Respect handling and sensitivity.** Intel reports may contain sensitive or attributable information; apply handling markings (TLP) and disseminate only to appropriate consumers. Over-dissemination of sensitive intel causes the same harms as over-sharing (the MISP/sharing skill).
7. **Close the feedback loop.** Ask whether the intelligence was useful and acted on; feedback refines both future reporting and the requirements. A programme that never checks whether its reports land can't improve.

### Cheatsheet

```
good analysis + bad communication = WASTED intelligence. the last mile matters.

write for the AUDIENCE + their DECISION (tactical/operational/strategic)
  SOC: concise, actionable indicators + detection content
  leadership: business-framed risk, plain language
  same analysis -> different reports per audience

LEAD with the answer (BLUF — bottom line up front)
  key judgement first (threat + impact + recommended action) ; analysis below for those who want it

ACTIONABLE: every report -> a decision/action (block / build detection / invest)
  no "so what / now what" = reader left to figure out the response

CONFIDENCE + sourcing: high/med/low + basis (overstating misleads ; hiding uncertainty = unusable)

DELIVER right form + timing
  tactical = fast + machine-consumable (indicators into tools, not a PDF)
  strategic = periodic briefing ; perishable intel delivered slowly = worthless

HANDLING (TLP): sensitive/attributable -> appropriate consumers only
FEEDBACK loop: was it useful + acted on? -> refine reporting + requirements
```

### Reading the reporting

- **A report that fits its audience and their decision** = intelligence that gets used; a SOC report of actionable indicators and an executive briefing of business risk both work because each fits its reader. The same analysis in the wrong form for the reader is wasted.
- **A report leading with analysis instead of the answer** = readers (especially busy executives) may never reach the key judgement; lead with the bottom line and put supporting analysis below.
- **A descriptive report with no recommended action** = leaves the reader to figure out the response; intelligence should drive a decision. State the "now what".
- **Overstated or hidden confidence** = misleads decisions either way; overconfidence leads to acting on shaky intel, vague hedging is unusable. Express calibrated confidence and the basis.
- **Perishable tactical intel delivered as a slow PDF** = worthless by the time it lands; tactical intel needs fast, machine-consumable delivery. Match form to shelf life.
- **A report that's useful, acted on, and fed back** = the programme working and improving; without a feedback loop, reporting can't get better and requirements drift.

### Pitfalls

- **Good analysis, poor communication.** The last mile wastes otherwise-valuable intelligence; a report nobody reads or can act on is a failure regardless of the analysis behind it.
- **Wrong form for the audience.** A technical report to executives or a strategic narrative to the SOC doesn't land; match the report to the consumer and their decision.
- **Burying the answer.** Leading with analysis instead of the bottom line means readers may never reach the key judgement. BLUF.
- **Non-actionable reports.** Describing a threat without a recommended action leaves the reader to work out the response; state the "so what / now what".
- **Mishandling confidence.** Overstating certainty misleads; hiding uncertainty in vague language is unusable. Calibrate and state confidence.
- **Slow delivery of perishable intel.** Tactical intel delivered too slowly or in the wrong form is worthless; match delivery to shelf life.
- **No feedback loop.** Not checking whether intel is used means the programme can't improve or realign to real needs.

### References

- The tactical-vs-strategic, intel-requirements, and mapping-intel-to-detection skills
- The vuln-mgmt reporting-to-stakeholders and GRC security-metrics-for-leadership skills
- BLUF and structured analytic writing (intelligence-community practice)
- TLP (Traffic Light Protocol) for handling/dissemination

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.