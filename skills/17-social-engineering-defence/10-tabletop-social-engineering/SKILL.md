---
name: tabletop-social-engineering
domain: 17-social-engineering-defence
description: Use when exercising the organisation's response to a social-engineering incident through a tabletop — walking through a realistic scenario to test people, process, and decisions before a real one hits.
difficulty: intermediate
tags: [social-engineering, tabletop, exercise, response, preparedness]
tools: []
---

## Purpose

A tabletop exercise walks key people through a realistic incident scenario in a discussion format — "here's what's happening, what do we do?" — to test the organisation's response before a real incident. For social engineering specifically, tabletops exercise the human and process response (who verifies the payment, who's authorised to decide, how fast the finance team escalates) that technical controls don't cover. This skill covers running a social-engineering tabletop, the low-cost way to find response gaps before an attacker does.

## When to use it

Testing preparedness for social-engineering incidents (BEC, a successful phish leading to account compromise, a vishing attack on the help desk) without the disruption of a live exercise. It complements phishing simulations (which test individual users) by testing the *organisational* response and decision-making.

## Procedure

1. **Choose a realistic, relevant scenario.** Base it on a plausible social-engineering incident for your organisation — a BEC wire-fraud attempt, an executive-impersonation payment request, a help-desk vishing that reset an MFA, a phishing-led account compromise. Relevance makes the exercise reveal real gaps; a far-fetched scenario tests nothing useful.
2. **Get the right people in the room.** Social-engineering response spans functions — security, IT, finance (for BEC/payment scenarios), HR, legal, comms, and leadership for decisions. The tabletop tests how they *coordinate*, so the people who'd actually be involved need to participate. A security-only tabletop misses the finance/decision gaps that matter most for social engineering.
3. **Walk through the scenario in stages, asking "what do we do?"** Present the situation as it would unfold ("finance received an urgent wire request from the CEO...") and have participants talk through their actions and decisions. Inject developments to test how the response adapts. The discussion surfaces who does what, who decides, and where the process is unclear.
4. **Test the specific social-engineering decision points** — the ones technical controls don't cover: does finance know to verify the payment out-of-band? who's authorised to approve/halt a transfer? how does the help desk verify identity before an MFA reset? how fast does a reported phish get investigated? These process/decision gaps are what the tabletop exists to find.
5. **Surface gaps without blame.** The point is finding weaknesses in process, roles, and decisions — not judging individuals. A blameless exercise gets honest participation and reveals the real gaps (an unclear escalation path, an undefined decision authority, a slow verification process).
6. **Capture findings and drive improvements.** Document the gaps found (missing runbook, unclear authority, slow reporting) and turn them into concrete fixes — process changes, defined decision authority, training. A tabletop that surfaces gaps but changes nothing wasted everyone's time.
7. **Run periodically and vary scenarios.** Preparedness decays and threats evolve; recurring tabletops with varied scenarios keep the response sharp and test different parts of the organisation.

## Cheatsheet

```
tabletop = discussion walk-through of a realistic incident ("what do we do?") — test response BEFORE real one
  for social engineering: tests HUMAN + PROCESS response (who verifies/decides/escalates) that tech doesn't cover
  low-cost way to find response gaps before an attacker does

run it
  1. REALISTIC RELEVANT scenario (BEC wire fraud / exec-impersonation payment / help-desk vishing MFA reset
     / phish-led account compromise) — far-fetched = tests nothing
  2. RIGHT PEOPLE: security + IT + FINANCE (BEC) + HR + legal + comms + LEADERSHIP (decisions)
     -> tests COORDINATION ; security-only misses the finance/decision gaps that matter most
  3. walk through in STAGES, "what do we do?" + inject developments -> surfaces who does what / who decides / unclear process
  4. test SE decision points (tech doesn't cover): out-of-band payment verification | transfer approve/halt authority
     | help-desk identity verification | reported-phish investigation speed
  5. BLAMELESS (find process/role/decision gaps, not judge people) -> honest participation
  6. CAPTURE findings -> concrete fixes (runbook, decision authority, training) ; gaps-but-no-change = wasted
  7. run periodically + vary scenarios
```

## Reading the exercise

- **A realistic, relevant scenario with the right cross-functional people** = the tabletop reveals real coordination and decision gaps; a security-only exercise on a far-fetched scenario finds nothing useful. Relevance and the right participants are what make it work.
- **Finance not knowing to verify a payment out-of-band, or unclear transfer-halt authority** = exactly the process/decision gaps a social-engineering tabletop exists to find; technical controls don't cover these, and they're where BEC succeeds. High-value findings.
- **An unclear escalation path or undefined decision authority** = surfaced by walking through "who decides?"; these coordination gaps cause slow, confused real responses. The tabletop makes them visible cheaply.
- **A blaming exercise** = people get defensive and honest gaps stay hidden; blameless discussion is what surfaces the real weaknesses. Keep it non-judgmental.
- **Gaps found but no changes made** = the exercise wasted everyone's time; the value is in turning findings into concrete fixes (runbooks, decision authority, training).
- **Recurring, varied tabletops driving process improvements** = preparedness maintained and gaps closed before a real incident tests them.

## Pitfalls

- **The wrong people in the room.** Social-engineering response spans finance, HR, legal, comms, and leadership — not just security. A security-only tabletop misses the finance/decision gaps that matter most for BEC and payment fraud. Get the cross-functional participants.
- **A far-fetched scenario.** It tests nothing useful; base the exercise on a plausible, relevant incident to reveal real gaps.
- **Focusing on technical response, missing the decision points.** The value for social engineering is the human/process decisions (out-of-band verification, transfer authority, help-desk verification) that technical controls don't cover. Test those.
- **Blaming individuals.** It kills honest participation and hides gaps; keep it blameless to surface the real weaknesses.
- **Not acting on findings.** A tabletop that surfaces gaps but drives no changes wasted the time; turn findings into concrete fixes.
- **Running it once.** Preparedness decays and threats change; run periodically with varied scenarios.

## References

- The IR incident-triage, communication-during-incidents, and ir-playbook-development skills (tabletops test the playbooks)
- The bec-detection, vishing-and-smishing-awareness, and reporting-culture skills (the scenarios)
- CISA tabletop exercise packages and NIST SP 800-84 (exercise guidance)
- The blameless-postmortem skill (same non-punitive discipline)
