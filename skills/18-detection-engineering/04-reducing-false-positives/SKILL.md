---
name: reducing-false-positives
domain: 18-detection-engineering
description: Use when a detection is too noisy — tuning it to cut false positives without creating blind spots, so alerts stay trustworthy and the SOC doesn't learn to ignore them.
difficulty: intermediate
tags: [detection, false-positives, tuning, alert-fatigue, quality]
tools: [sigma]
---

## Purpose

A detection that cries wolf is worse than no detection — the SOC learns to ignore it, and the one time it's real gets closed as noise. But the naive fix, suppressing aggressively, creates the opposite failure: blind spots where real attacks pass silently. Reducing false positives well means cutting the noise while keeping the detection's ability to catch the real thing. This skill covers that balance, the core craft of maintaining a healthy detection.

## When to use it

When a detection generates too many false positives (revealed by SOC feedback, triage data, or the detection-metrics skill flagging noisy rules). It's ongoing maintenance — detections that were precise drift noisy as the environment changes, and the loudest rules usually generate most of the fatigue.

## Procedure

1. **Understand *why* it false-positives before touching it.** Look at the actual false-positive alerts: what benign activity is matching? A backup job, an admin tool, a legitimate scanner, a specific application's normal behaviour. The tuning follows from the cause — blind suppression without understanding is how you create blind spots.
2. **Tune with precision, not a blunt off-switch.** Options in order of preference:
   - **Add context to the logic** — narrow the rule so it matches the malicious pattern more precisely (e.g. the suspicious command *from an unexpected parent process*, not the command alone).
   - **Exclude known-good specifically** — filter the exact benign source (that backup account, that admin host), using `selection and not filter`, so you subtract the false positives without widening the blind spot.
   - **Baseline/allowlist the environment's normal** — legitimate-but-unusual activity specific to your org.
3. **Don't over-suppress — the opposite failure.** Every exclusion is a potential blind spot: an attacker who operates from the excluded source or mimics the allowlisted pattern goes undetected. Keep exclusions as narrow and specific as possible, and track what you've excluded so you know your gaps.
4. **Preserve the detection's intent.** After tuning, confirm the rule still fires on the actual malicious behaviour (re-test against known-malicious — the testing skill). A tuned rule that no longer catches the attack has been tuned to death.
5. **Prefer enrichment over suppression where possible.** Sometimes the fix isn't to silence the rule but to add context (asset criticality, user role) so the analyst can triage faster, keeping the detection while cutting the *effort* per alert (the alert-enrichment skill).
6. **Iterate and document.** Tuning is rarely one-shot; deploy the change, watch the false-positive rate, refine. Record each exclusion and why, so the rule stays maintainable and the blind spots are known.

## Cheatsheet

```
FIRST: understand WHY it false-positives (read the actual FP alerts)
  what benign thing matches? backup / admin tool / scanner / app behaviour
  -> tune the CAUSE, not blindly

tune with precision (preferred order)
  1. add context to logic   suspicious cmd + unexpected PARENT (not cmd alone)
  2. exclude known-good specifically   selection AND NOT filter (exact benign source)
  3. baseline/allowlist env normal      narrowly

DON'T over-suppress (opposite failure = blind spots)
  every exclusion = potential blind spot (attacker uses the excluded source/pattern)
  keep exclusions NARROW + specific ; TRACK what you excluded (= your gaps)

preserve intent: re-test -> still fires on the real malicious behaviour?
  (tuned so it no longer catches the attack = tuned to death)

prefer ENRICHMENT over suppression where possible (keep rule, cut effort/alert)
iterate: deploy -> watch FP rate -> refine ; document every exclusion + why
```

## Reading the tuning

- **A rule generating mostly false positives** = alert fatigue; the SOC will start ignoring it, and a real hit gets closed as noise. The most important rules to tune are the loudest. Understand the cause and narrow it.
- **The false-positive cause identified** (a specific backup account, an admin tool) = you can tune precisely — exclude that exact source or add the distinguishing context — rather than blunt-suppressing.
- **A broad exclusion that silences the noise** = check it didn't create a blind spot; if you excluded a whole host or user, an attacker operating from there is now invisible. Narrow it.
- **A tuned rule that no longer fires on the real attack** = over-tuned; you traded false positives for a false negative, which is worse. Re-test to confirm it still catches the malicious behaviour.
- **A rule where enrichment (not suppression) would help** = keep the detection and add context so triage is faster; you cut the cost without losing the coverage.
- **A precisely-tuned rule that's quiet on benign and fires on malicious, with documented exclusions** = the healthy balance this skill aims for.

## Pitfalls

- **Blind suppression.** Silencing a noisy rule without understanding what's matching creates blind spots — you may have excluded exactly where an attacker operates. Understand the cause first.
- **Over-tuning into false negatives.** Tuning so aggressively that the rule no longer catches the real attack is worse than the noise; a missed attack beats a false positive. Re-test after tuning.
- **Broad exclusions.** Excluding a whole user/host/subnet to kill noise opens a wide blind spot. Keep exclusions as specific as possible and track them.
- **Ignoring alert fatigue.** A noisy rule left noisy trains the SOC to ignore it, so the real alert dies in the noise. Tune the loud rules — they cause most of the fatigue.
- **Tuning without tracking.** Undocumented exclusions become unknown blind spots; record what you excluded and why.
- **Suppressing when enrichment would do.** Sometimes the alert is valid but hard to triage; adding context beats silencing it.

## References

- The writing-sigma-rules, testing-detections, alert-enrichment, and detection-metrics skills
- The SOC alert-triage-workflow and network ids-ips-tuning skills (same tuning discipline)
- SANS detection engineering / SOC tuning resources
