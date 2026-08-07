---
name: reporting-and-retest
domain: 16-red-teaming-and-adversary-emulation
description: Use when writing up a red-team engagement and verifying fixes — producing findings the organisation can actually close, and re-testing to confirm remediation worked.
difficulty: intermediate
tags: [red-team, reporting, retest, remediation, findings]
tools: []
---

## Purpose

An engagement's value is realised only in the report and the remediation it drives — a brilliant intrusion that produces an unactionable report improved nothing. This skill covers writing red-team findings the organisation can close, and re-testing to confirm the fixes worked. It's the closing discipline that turns offensive work into defensive improvement, and it's where a lot of red-team value is won or lost.

## When to use it

At the end of every engagement, and after the organisation remediates. Reporting well is what makes the engagement worthwhile; re-testing is what confirms the money and effort actually reduced risk.

## Procedure

1. **Write for the audiences — technical and executive.** Like other security reporting, a red-team report serves different readers: technical teams need the detailed findings and fixes; leadership needs the risk picture and business impact. Lead with an executive summary (what was achieved, what it means for the business, key recommendations), with technical detail below.
2. **Tell the attack narrative, mapped to ATT&CK.** The strength of a red-team report is the story: how the emulated adversary got in, escalated, moved, and achieved the objective — the full attack path, mapped to ATT&CK techniques. This narrative shows the organisation exactly how an intrusion would unfold against them.
3. **Make findings actionable — the core.** Every finding needs: what the weakness is, how it was exploited (the specific path), the impact, and — critically — a concrete, prioritised remediation. A finding without a clear fix leaves the reader to figure out the response. Prioritise by risk so the organisation fixes the highest-impact issues first.
4. **Include the defensive outcome — what was detected.** A red-team report isn't just "here's what got through"; it should say what the blue team *did* detect and where detection failed. This is often as valuable as the attack findings — it tells the organisation where their detection and response worked and where they didn't (feeding the detection/SOC domains).
5. **Emphasise systemic fixes over point fixes.** Where findings share a root cause (flat network enabling lateral movement, missing tiering enabling escalation), recommend the systemic fix that closes the class, not just the individual instances. Breaking the attack path at a chokepoint is more valuable than patching each step.
6. **Re-test after remediation — confirm the fixes worked.** A remediation that's reported as done isn't proven until verified; re-test the specific findings to confirm the fixes actually close them (the vuln-management remediation-verification discipline). Re-testing is what turns "we fixed it" into "it's actually fixed", and it's a standard part of a quality engagement.
7. **Feed lessons into the defence.** The findings should drive detection improvements (the gaps found), hardening (the weaknesses exploited), and architecture (the systemic issues) — and ideally the engagement's techniques become detection tests (purple-teaming / detection domain). The engagement improves the defence, or it was just an expensive intrusion.

## Cheatsheet

```
engagement value = the REPORT + the remediation it drives (great intrusion + unactionable report = nothing improved)

report
  AUDIENCES: technical (findings + fixes) + executive (risk, business impact, key recs)
    -> lead with executive summary, technical detail below
  ATTACK NARRATIVE mapped to ATT&CK (how they got in -> escalated -> moved -> objective — the full path)
  ACTIONABLE findings (core): weakness + how exploited (path) + impact + CONCRETE prioritised remediation
    (no fix = reader left guessing ; prioritise by risk)
  DEFENSIVE OUTCOME: what was DETECTED + where detection failed (as valuable as the attack findings)
  SYSTEMIC fixes > point fixes (shared root cause -> fix the class ; break the path at a chokepoint)

RE-TEST after remediation: reported-done != proven -> verify the specific findings are closed
  (turns "we fixed it" -> "it's actually fixed" ; standard part of a quality engagement)

feed lessons -> detection improvements + hardening + architecture (+ techniques -> detection tests)
  the engagement IMPROVES the defence, or it was an expensive intrusion.
```

## Reading the deliverable

- **Findings with concrete, prioritised remediations** = an actionable report the organisation can close; the core of red-team value. A finding without a clear fix leaves the reader to work out the response and often goes unremediated.
- **An unactionable report** (a narrative of the intrusion with no clear fixes) = the engagement's value largely lost; a great intrusion that improves nothing. Actionability is what matters.
- **The attack narrative mapped to ATT&CK** = shows exactly how an intrusion unfolds against the organisation and lets them compare to their detection coverage; the report's storytelling strength.
- **The defensive outcome included** (what was detected, where detection failed) = often as valuable as the attack findings; it tells the organisation where their detection/response worked. A report that omits this misses half the value.
- **Systemic fixes recommended over point fixes** = closing classes of weakness (fix the flat network, not each movement instance) is more valuable; breaking the path at a chokepoint stops the whole chain.
- **Re-test confirming fixes closed the findings** = remediation proven, not just claimed; the engagement's risk reduction is verified. Without re-test, "fixed" is unverified.

## Pitfalls

- **An unactionable report.** A brilliant intrusion with vague or missing fixes improves nothing; every finding needs a concrete, prioritised remediation. Actionability is where red-team value is won or lost.
- **Omitting the defensive outcome.** A report that's only "what got through" misses what the blue team detected and where detection failed — often as valuable as the attack findings. Include it.
- **Point fixes over systemic ones.** Recommending a fix per finding while missing the shared root cause leaves the attack path open through other instances; recommend the systemic fix that closes the class.
- **No re-test.** Reported-remediated isn't proven-remediated; without re-testing, the risk reduction is unverified and fixes may not actually close the findings.
- **One report for all audiences.** Technical teams and leadership need different things; lead with an executive summary and provide technical depth below.
- **Not feeding lessons into the defence.** If the findings don't drive detection, hardening, and architecture improvements, the engagement was an expensive intrusion. Close the loop.

## References

- The vulnerability-management remediation-verification and reporting-to-stakeholders skills
- The purple-teaming skill (techniques → detection tests) and the detection-engineering domain
- MITRE ATT&CK (narrative mapping), PTES (reporting), and NIST SP 800-115
- The scoping-and-rules-of-engagement skill (objectives → report success criteria)
