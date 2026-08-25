---
format: "v2"
name: "tactical-vs-strategic"
title: "Tactical Vs Strategic"
title_fr: "Renseignement tactique vs stratégique"
description: "Use when producing intelligence for different audiences — understanding the tactical, operational, and strategic levels so intel reaches the SOC, IR, and leadership in the form each needs."
description_fr: "À utiliser pour produire du renseignement adapté à différents publics — comprendre les niveaux tactique, opérationnel et stratégique afin que le renseignement atteigne le SOC, la réponse à incident et la direction sous la forme dont chacun a besoin."
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

Threat intelligence serves very different consumers — the SOC analyst blocking an indicator, the IR team understanding a campaign, and the executive deciding on security investment — and each needs intel in a completely different form. Producing intelligence without understanding these levels means the SOC drowns in strategic reports it can't action and leadership gets indicator lists it can't understand. This skill covers the tactical/operational/strategic levels of intelligence and matching each to its audience, so intel actually gets used.

### When to use it

When producing or organising threat intelligence for consumption, and when a programme's intel isn't landing (the SOC ignores it, leadership doesn't get it). It's the framing that makes an intel programme serve its whole organisation rather than one audience.

### The three levels

- **Tactical** — the immediate, technical detail: indicators (IPs, domains, hashes), specific TTPs, and detection/blocking content. **Audience:** SOC, detection engineers, IR. **Use:** feed directly into detection, blocking, and hunting. Perishable and specific. Timescale: now.
- **Operational** — intelligence about campaigns and adversary operations: who's active, what campaigns are running, how an actor operates against your sector. **Audience:** IR, threat hunters, security management. **Use:** anticipate and prepare for specific threats, prioritise defences. Timescale: weeks to months.
- **Strategic** — the big picture: threat landscape trends, risks to the business, geopolitical/sector context, and what it means for the organisation. **Audience:** leadership, executives, the board. **Use:** inform security investment, risk decisions, and strategy. Non-technical, business-framed. Timescale: months to years.

### Procedure

1. **Identify who consumes your intelligence and what decision each makes.** The SOC decides "block/alert on this"; IR decides "how do I respond to this campaign"; leadership decides "where do we invest". The level of intel follows from the decision it supports.
2. **Match the level to the audience — the core principle.** Deliver tactical intel (indicators, TTPs, detection content) to the SOC/IR in machine-consumable form; operational intel (campaigns, actor activity) to hunters and security management; strategic intel (trends, business risk) to leadership in plain language. Mismatching wastes the intel.
3. **Translate up and down the levels.** The same underlying threat produces intel at all three levels — an indicator (tactical), the campaign it belongs to (operational), the trend it represents (strategic). Good intel programmes translate a threat into the right form for each audience rather than producing one form for all.
4. **Frame strategic intel in business terms.** Leadership doesn't need CVEs and hashes; they need "our sector is being targeted by ransomware groups, our exposure is X, here's the recommended investment". Strategic intel that stays technical doesn't inform business decisions (ties into the reporting skill).
5. **Keep tactical intel machine-consumable and timely.** The SOC needs indicators and detection content in a form that flows into tools automatically and fast (perishable), not prose reports they have to read and transcribe.
6. **Don't send the wrong level to the wrong audience.** Sending strategic reports to the SOC (unactionable) or indicator dumps to executives (incomprehensible) are the classic failures; each audience gets the level it can use.
7. **Drive production from requirements.** What each audience needs (intelligence requirements — that skill) should drive what you produce at each level, rather than producing whatever's available and hoping it's useful.

### Cheatsheet

```
different consumers need intel in DIFFERENT forms — match level to audience or it's wasted

level        content                          audience              use / timescale
-----------  -------------------------------  --------------------  ---------------------
TACTICAL     indicators, specific TTPs,        SOC, detection eng,   feed detection/blocking/hunt
             detection content (perishable)    IR                    / NOW (machine-consumable, fast)
OPERATIONAL  campaigns, adversary operations,  IR, hunters,          anticipate/prepare, prioritise
             actor activity vs your sector     security mgmt         / weeks-months
STRATEGIC    threat landscape, business risk,  leadership, board     investment, risk, strategy
             geopolitical/sector context       (non-technical!)      / months-years

core: match level to audience + the DECISION it supports
translate one threat into all 3 forms (indicator -> campaign -> trend)
strategic = BUSINESS terms (not CVEs/hashes) ; tactical = machine-consumable + timely
classic failures: strategic report -> SOC (unactionable) ; indicator dump -> exec (incomprehensible)
drive from REQUIREMENTS (what each audience needs), not what's available
```

### Reading intel production

- **The SOC getting strategic reports** = a mismatch; they can't action a threat-landscape narrative. They need tactical indicators and detection content in machine-consumable form. Sending the wrong level wastes the intel and trains the audience to ignore it.
- **Leadership getting indicator lists** = the opposite mismatch; executives can't act on IPs and hashes. Strategic intel must be business-framed (exposure, risk, investment), not technical.
- **One threat translated into all three levels** = a mature programme; the indicator feeds the SOC, the campaign informs IR/hunters, the trend informs leadership — the same threat serving every audience in its own form.
- **Tactical intel in prose reports** = too slow and un-consumable for the SOC; tactical needs to flow into tools automatically and fast, because it's perishable.
- **Strategic intel that stays technical** = doesn't inform business decisions; leadership needs the "what it means for us" translation, not the technical detail.
- **Intel production driven by audience requirements** = intel that lands; producing whatever's available and hoping misses what each audience actually needs.

### Pitfalls

- **Mismatching level and audience.** The core failure — strategic reports to the SOC (unactionable) or indicators to executives (incomprehensible). Match the level to who consumes it and the decision they make.
- **Producing only one level.** A programme that only does tactical (indicators) never informs leadership; one that only does strategic never helps the SOC. Cover the levels your audiences need.
- **Keeping strategic intel technical.** Leadership needs business framing, not CVEs; technical strategic intel doesn't drive investment decisions.
- **Slow/unconsumable tactical intel.** The SOC needs indicators fast and machine-readable; prose reports of perishable tactical intel arrive too late and in the wrong form.
- **Producing what's available, not what's needed.** Intel production should follow audience requirements, not push whatever data exists and hope it's useful.

### References

- The intel-requirements, reporting-and-dissemination, and mapping-intel-to-detection skills
- The GRC security-metrics-for-leadership skill (strategic framing)
- Standard threat-intelligence level definitions (tactical/operational/strategic)
- SANS and industry CTI (cyber threat intelligence) frameworks

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.