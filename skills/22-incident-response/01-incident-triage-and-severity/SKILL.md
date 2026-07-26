---
name: incident-triage-and-severity
domain: 22-incident-response
description: Use at the start of a suspected security incident to decide quickly whether it's real, how bad it is, and what response it warrants — before jumping to containment.
difficulty: intermediate
tags: [incident-response, triage, severity, soc]
tools: []
---

## Purpose

The first minutes of an incident set the tone for everything after. Move too slow and damage spreads; move too fast on a false alarm and you burn the team on nothing. This skill is a repeatable way to answer three questions quickly: is this a real incident, how severe is it, and who needs to be pulled in — before you touch containment.

## When to use it

The moment an alert, report, or anomaly looks like it might be a security incident. It sits before every other IR skill; severity decided here drives the containment strategy, the comms, and whether you're declaring a formal incident at all.

## Procedure

1. **Confirm it's real.** Rule out the benign explanations first — a scheduled job, a pentest in progress, a known-good admin action, a misconfigured monitor. Check whether anything is genuinely anomalous versus expected-but-noisy.
2. **Scope it, roughly.** You don't need the full picture, just enough to size it: one host or many? One account or a spread? Data involved? Internet-facing? Note what you know and, explicitly, what you don't.
3. **Assign severity** against a simple, pre-agreed scale rather than gut feeling. Anchor on impact and spread:
   - **Critical** — active compromise of production/sensitive data, ransomware detonating, domain-level compromise, live data exfiltration. All hands.
   - **High** — confirmed compromise of a significant system or account, contained but not resolved.
   - **Medium** — a real but limited issue (single non-critical host, blocked attempt with signs of persistence).
   - **Low** — minor or likely-benign, worth logging and watching.
4. **Decide the response tier** from the severity: who's the incident lead, is this a formal declaration, who gets paged now vs updated later, what's the meeting cadence.
5. **Preserve while you triage.** Note timestamps, alert IDs, and observations, and avoid actions that destroy volatile evidence (don't reboot, don't wipe) — real containment and imaging come next, in their own skills.
6. **Hand off cleanly.** Triage output is a short, factual brief: what happened, current severity, scope so far, what's confirmed vs assumed, and the immediate next step.

## Cheatsheet

```
triage questions, in order
  1. real? ......... benign cause ruled out?
  2. scope? ........ hosts / accounts / data / exposure
  3. severity? ..... Critical / High / Medium / Low (agreed scale)
  4. response? ..... lead named, declare y/n, who to page
  5. preserve? ..... timestamps logged, no destructive actions
  6. handoff ....... short factual brief to the responders

severity anchors
  Critical  active prod/data compromise, ransomware, domain owned
  High      confirmed compromise, significant system/account
  Medium    real but limited, single non-critical host
  Low       minor / likely benign, monitor
```

## Reading the situation

- **Multiple hosts or accounts showing the same pattern** = spread; bias severity up and think lateral movement.
- **Anything touching production data, credentials, or domain infrastructure** is Critical/High until proven otherwise — err upward early, downgrade later with evidence.
- **A single blocked attempt with no follow-through** is usually Low/Medium — but check for persistence before you close it.
- **"We can't tell yet"** is a valid triage state. Assign a provisional severity, say what would raise or lower it, and keep moving — don't stall waiting for certainty.

## Pitfalls

- **Skipping straight to containment.** Pulling the wrong plug on a misread alarm wastes the team and can destroy evidence. Confirm and size first — but don't over-invest in triage while something is actively burning; for a clear Critical, triage is fast and you move.
- **Anchoring severity on the alert's label.** A "critical" signature can be a false positive; a "low" one can be the first sign of a breach. Rate by actual impact and scope.
- **Under-calling to avoid the hassle of declaring.** Downplaying a real incident to dodge escalation is how small incidents become big ones.
- **Destroying volatile evidence.** Rebooting or "cleaning" a host during triage throws away memory and live artefacts the forensics skill needs.

## References

- NIST SP 800-61r2 (Computer Security Incident Handling Guide)
- SANS Incident Handler's Handbook
- FIRST — CVSS and incident severity guidance
