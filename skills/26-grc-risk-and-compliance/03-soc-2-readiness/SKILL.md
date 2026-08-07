---
name: soc-2-readiness
domain: 26-grc-risk-and-compliance
description: Use when preparing for a SOC 2 audit — understanding the Trust Services Criteria, Type I vs Type II, and getting the controls and evidence in place to pass without last-minute panic.
difficulty: intermediate
tags: [grc, soc-2, compliance, audit, trust-services]
tools: []
---

## Purpose

SOC 2 is an attestation report, produced by an auditor, on how well an organisation's controls meet the Trust Services Criteria — most commonly required by B2B/SaaS customers as proof their vendor is secure. Preparing for SOC 2 means having the right controls in place *and* the evidence to prove they operated over time. This skill covers SOC 2 readiness — understanding what it requires and preparing so the audit confirms real practice rather than a scramble to fake it.

## When to use it

When customers or contracts require a SOC 2 report (common for SaaS/B2B vendors), or preparing for the audit. Understanding Type I vs Type II and the evidence requirement early prevents the classic last-minute panic.

## Procedure

1. **Understand what SOC 2 is — an auditor's attestation, not a certification.** An independent auditor (CPA firm) examines your controls against the Trust Services Criteria and issues a report. It's not pass/fail like ISO certification; the report describes the controls and the auditor's opinion, and customers read it to assess your security.
2. **Know the Trust Services Criteria (TSC).** SOC 2 covers up to five categories: **Security** (mandatory — the common criteria), **Availability**, **Processing Integrity**, **Confidentiality**, and **Privacy**. Most organisations start with Security; add others based on what customers need and what's relevant. Scope to the criteria that matter.
3. **Understand Type I vs Type II — a key decision.** **Type I** attests that controls are *designed* appropriately at a point in time (a snapshot). **Type II** attests that controls *operated effectively over a period* (typically 3–12 months) — it requires evidence the controls actually ran throughout. Type II is what most customers want (it proves ongoing operation), and it means you need a period of evidence, not a one-time setup. This drives the timeline.
4. **Implement the controls the TSC require** — access controls, change management, monitoring, incident response, risk management, vendor management, etc. These map to controls you likely have from other frameworks (the control-mapping skill); SOC 2 is less prescriptive than ISO about *which* controls, more about demonstrating they work.
5. **Collect evidence continuously — the make-or-break for Type II.** Type II requires proof the controls operated over the whole period — access reviews performed, changes approved, incidents handled, monitoring alerts triaged. Collect this evidence *as it happens* throughout the period, not at the end. The classic failure is realising at audit time you have no evidence the controls ran for the past six months. Evidence-and-audit-prep (that skill) is central.
6. **Do a readiness assessment / gap analysis first.** Before the real audit, assess where you stand against the TSC (the gap-analysis skill) and fix gaps — going into the audit with known gaps wastes money and produces a bad report.
7. **Engage the auditor and manage the timeline.** For Type II, the observation period must pass with evidence before the auditor can attest; plan the timeline backward from when you need the report (customers asking now means you needed to start months ago).

## Cheatsheet

```
SOC 2 = an AUDITOR's attestation on controls vs Trust Services Criteria (not pass/fail cert)
  customers (B2B/SaaS) read the report to assess vendor security

TRUST SERVICES CRITERIA (up to 5): SECURITY (mandatory) + Availability + Processing Integrity
  + Confidentiality + Privacy — scope to what customers need

TYPE I vs TYPE II (key decision -> drives timeline)
  Type I  = controls DESIGNED right at a POINT in time (snapshot)
  Type II = controls OPERATED effectively over a PERIOD (3-12mo) — needs a PERIOD of evidence
    -> most customers want Type II (proves ongoing operation)

prepare
  implement TSC controls (access, change mgmt, monitoring, IR, risk, vendor — map from other frameworks)
  COLLECT EVIDENCE CONTINUOUSLY (Type II make-or-break): access reviews, change approvals, incidents,
    alert triage — AS IT HAPPENS all period, NOT at the end
    (classic failure: audit time + no evidence controls ran for 6 months)
  READINESS/GAP ASSESSMENT first (fix gaps before the real audit)
  engage auditor + manage TIMELINE (Type II: observation period must pass first ; plan backward)
```

## Reading readiness

- **Realising at audit time you have no evidence the controls operated over the period** = the classic Type II failure; the controls may exist but you can't prove they ran. Continuous evidence collection throughout the period is make-or-break — this is where readiness is won or lost.
- **Confusing Type I and Type II** = a timeline problem; Type II needs a 3–12 month observation period with evidence *before* the auditor can attest. If customers need Type II now and you're starting now, you're months behind. Understand this early.
- **Going into the audit with known gaps** = wasted money and a bad report; a readiness/gap assessment first (the gap-analysis skill) lets you fix gaps before the real audit.
- **Controls that exist but aren't evidenced** = insufficient for Type II; the audit examines evidence of operation, not just control design. Evidence is as important as the control.
- **Scoping to only the relevant Trust Services Criteria** = efficient; Security is mandatory, others by customer need. Don't over-scope.
- **Controls implemented, evidence collected continuously, gaps closed pre-audit, timeline planned** = SOC 2 readiness that produces a clean report reflecting real practice.

## Pitfalls

- **No continuous evidence for Type II.** The make-or-break failure — realising at audit time you can't prove controls operated over the period. Collect evidence as it happens throughout, not at the end.
- **Misunderstanding Type I vs Type II timing.** Type II needs a period of evidence before attestation; if you need a Type II report now, you needed to start months ago. Plan the timeline backward.
- **Going in with known gaps.** A readiness assessment first, fixing gaps, avoids a wasted audit and a poor report.
- **Treating it as a certification.** SOC 2 is an auditor's attestation/report, not pass/fail; customers read the report and its exceptions. The report reflects reality.
- **Over-scoping the criteria.** Security is mandatory; add others only as customers need. Don't attest to criteria that don't apply.
- **Controls without evidence.** The audit examines evidence of operation; a control that isn't evidenced doesn't count for Type II.

## References

- AICPA Trust Services Criteria and SOC 2 guidance
- The gap-analysis, control-mapping, and evidence-and-audit-prep skills
- SOC 2 readiness / compliance automation tooling (evidence collection)
- The iso-27001-isms skill (overlapping controls, different framework)
