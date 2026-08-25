---
format: "v2"
name: "gap-analysis"
title: "Gap Analysis"
title_fr: "Analyse d'écarts"
description: "Use when measuring the distance between where your security is and where it needs to be — against a framework, a target state, or a requirement — and turning the gaps into a prioritised roadmap."
description_fr: "À utiliser pour mesurer la distance entre l'état actuel de votre sécurité et l'état visé — face à un référentiel, un état cible ou une exigence — et transformer les écarts en feuille de route priorisée."
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

A gap analysis measures the distance between your current security state and a target — a framework's requirements, a maturity level, a regulatory obligation, or a desired posture — and identifies what's missing. It's how you know what to work on, turn "we should be more secure" into a concrete list of gaps, and build a prioritised roadmap. This skill covers running a gap analysis that produces actionable improvement, not just a report of everything you don't have. It's a natural closing skill — the whole GRC domain (and much of security) is about closing the gap between current and required.

### When to use it

Before pursuing a certification (where do we stand vs ISO 27001 / SOC 2?), assessing security maturity, planning a security programme, or meeting a new requirement (the gap between current state and NIS2, say). It underpins compliance readiness (the soc-2 and iso skills) and programme planning.

### Procedure

1. **Define the target state clearly.** A gap analysis is meaningless without a defined "should be" — a framework (ISO 27001 controls), a maturity model (CMMI-style levels), a regulation's requirements, or a documented target posture. The target is the yardstick; a vague target produces a vague analysis. Choose the right target for the goal.
2. **Assess the current state honestly.** Evaluate what you actually have against each element of the target — controls in place, their maturity, evidence of operation. Honesty is essential: a gap analysis that overstates the current state (to look good) hides the gaps you need to fix and misleads planning. Assess reality, not aspiration.
3. **Identify the gaps.** For each target element, the gap is the difference — missing entirely, partially implemented, implemented but not operating/evidenced, or fully met. Categorise gaps by type (missing control, immature control, missing evidence), since the fix differs.
4. **Prioritise the gaps — the key to actionability.** A raw list of every gap is overwhelming and not a plan. Prioritise by risk (which gaps expose the most risk — tie to the risk assessment), by requirement (which are mandatory for a certification/regulation deadline), and by effort (quick wins vs major projects). Prioritisation turns a gap list into a roadmap.
5. **Build a roadmap, not just a report.** The output should be a prioritised, sequenced plan to close the gaps — what to fix, in what order, with owners and timelines — not a static list of deficiencies. A gap analysis that produces a report nobody acts on wasted the effort; the value is the improvement it drives.
6. **Distinguish must-fix from should-improve.** Not every gap is equal — a mandatory-control gap blocking certification is different from a maturity improvement. Separate the gaps you *must* close (for a requirement/deadline or high risk) from the ones that are improvements to schedule.
7. **Re-run periodically to track progress.** A gap analysis is a point-in-time measure; re-running it shows progress (gaps closed) and surfaces new gaps as the target (frameworks, threats) evolves. It's a recurring instrument for tracking a programme's maturation, not a one-time exercise.

### Cheatsheet

```
gap analysis = distance between CURRENT state and a TARGET (framework / maturity / regulation / desired posture)
  -> what to work on ; turns "be more secure" into concrete gaps + a prioritised ROADMAP
  (the whole GRC domain = closing current->required)

do
  1. DEFINE TARGET clearly (framework/maturity model/regulation/documented posture) — the yardstick
     (vague target = vague analysis)
  2. ASSESS CURRENT honestly (controls, maturity, evidence) — overstating hides the gaps you must fix
  3. IDENTIFY gaps: missing / partial / implemented-but-not-operating-or-evidenced / met
     (categorise by type — fix differs)
  4. PRIORITISE (key to actionability): by RISK (risk assessment) + REQUIREMENT (mandatory/deadline) + EFFORT (quick wins)
     -> raw list = overwhelming ; prioritisation = a roadmap
  5. build a ROADMAP not a report (prioritised sequenced plan + owners + timelines ; report nobody acts on = wasted)
  6. MUST-FIX vs should-improve (mandatory/high-risk gap vs maturity improvement — separate them)
  7. RE-RUN periodically (progress + new gaps as target evolves) — recurring instrument, not one-time
```

### Reading the analysis

- **A gap analysis with no clearly-defined target** = meaningless; "gaps against what?" A vague target (or none) produces a vague, unactionable analysis. Define the yardstick first.
- **A current-state assessment that overstates what you have** = hides the gaps you need to fix and misleads planning; honesty is essential. A gap analysis that flatters the current state defeats its purpose.
- **A raw list of every gap** = overwhelming and not a plan; prioritisation by risk, requirement, and effort is what turns it into an actionable roadmap. The list without prioritisation is a wall of deficiencies.
- **A report of gaps that nobody acts on** = wasted effort; the value is the improvement it drives, so the output must be a prioritised, owned, sequenced roadmap. A static deficiency report changes nothing.
- **Must-fix gaps not separated from improvements** = a certification-blocking mandatory gap gets the same weight as a nice-to-have maturity item; distinguish what you must close (deadline/high-risk) from what to schedule.
- **A defined-target, honestly-assessed, prioritised, roadmap-producing, periodically-re-run gap analysis** = the instrument that drives a security programme from where it is to where it needs to be — closing the current-to-required gap that security is fundamentally about.

### Pitfalls

- **No clearly-defined target.** A gap analysis needs a "should be" to measure against; without a clear target (framework, maturity, regulation), the analysis is vague and unactionable. Define the yardstick.
- **Overstating the current state.** Flattering your current posture hides the gaps you need to fix and misleads planning; assess honestly, reality not aspiration.
- **A raw, unprioritised gap list.** It's overwhelming and not a plan; prioritise by risk, requirement, and effort to produce a roadmap. Prioritisation is what makes it actionable.
- **Producing a report, not a roadmap.** A static list of deficiencies nobody acts on wasted the effort; the value is the sequenced, owned improvement plan it drives.
- **Not distinguishing must-fix from improvements.** Mandatory/high-risk gaps and maturity improvements need different treatment; separate them.
- **One-time analysis.** Targets and threats evolve; re-run periodically to track progress and surface new gaps. It's a recurring instrument.

### References

- The risk-assessment (prioritisation), iso-27001-isms, and soc-2-readiness skills (the targets)
- The control-mapping skill (mapping current controls to target requirements)
- Maturity models (NIST CSF tiers, CMMI, C2M2) as target states
- NIST CSF and ISO 27001 as common gap-analysis frameworks

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.