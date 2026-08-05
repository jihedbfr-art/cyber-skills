---
name: detection-as-code
domain: 18-detection-engineering
description: Use when managing detections like software — version control, testing, and CI/CD for detection rules so they're reviewed, tested, and deployed reliably instead of edited in a console.
difficulty: intermediate
tags: [detection, detection-as-code, cicd, version-control, testing]
tools: [git, sigma, ci]
---

## Purpose

Detections are logic that decides whether you catch an attack — too important to be edited ad hoc in a SIEM console with no history, review, or testing. Detection-as-code applies software engineering practice to detection rules: they live in version control, get reviewed, get tested automatically, and deploy through a pipeline. This skill covers running detections that way, which turns a fragile pile of console rules into a maintainable, trustworthy detection library.

## When to use it

Building or maturing a detection-engineering practice beyond a handful of hand-edited rules. It's the operational backbone that makes the other detection skills (Sigma writing, testing, tuning) repeatable and safe at scale.

## Procedure

1. **Put detections in version control.** Store rules (Sigma, SIEM-native, EDR) as files in a git repository — the single foundational move. Now every change has history, an author, and a diff; you can see who changed a rule, when, and why, and roll back a bad change.
2. **Require review.** Detection changes go through pull requests reviewed by another engineer, like code. Review catches broken logic, false-positive risk, and missing coverage before a rule reaches production and pages someone at 3am.
3. **Test automatically in CI.** A pipeline validates each change on commit — syntax/schema validation, and ideally functional testing (does the rule fire on known-malicious samples, and stay quiet on benign — see the testing-detections skill). Untested rules are hypotheses; CI makes testing non-optional.
4. **Deploy through a pipeline, promoting by maturity.** Rules move from experimental → test → production automatically as they pass validation, so unvetted rules don't page anyone and deployment is consistent rather than a manual console edit that drifts.
5. **Keep detections portable and structured.** Author in a portable format (Sigma) where possible and convert to the target platform in the pipeline, so the logic isn't locked to one SIEM and can be tested and reused.
6. **Track detections against coverage** (ATT&CK mapping in the rule metadata) so the repository doubles as your coverage map (feeds the metrics and mapping skills).
7. **Treat the whole thing like a codebase** — issues for gaps, branches for work, releases, and documentation. The detection library becomes a managed engineering artifact, not tribal knowledge in a console.

## Cheatsheet

```
detections = code -> version control + review + test + pipeline

1. VERSION CONTROL   rules as files in git (Sigma / SIEM-native / EDR)
                     -> history, authorship, diffs, rollback (the foundational move)
2. REVIEW            changes via pull request, reviewed by another engineer
                     -> catch broken logic / FP risk / gaps BEFORE production
3. CI TESTING        on commit: schema/syntax validate + functional test
                     (fires on malicious? quiet on benign?) -> testing non-optional
4. PIPELINE DEPLOY   promote experimental -> test -> production by maturity
                     -> unvetted rules don't page ; consistent deploy (no console drift)
5. PORTABLE          author in Sigma, convert per-platform in pipeline
6. COVERAGE          ATT&CK mapping in metadata -> repo = coverage map
7. manage it like a codebase: issues, branches, releases, docs

vs the anti-pattern: rules hand-edited in the SIEM console, no history/review/test.
```

## Reading the practice

- **Detections hand-edited in the console with no history** = the anti-pattern this fixes; no way to know who changed what, no review, no rollback, and rules drift silently. Move them into version control.
- **Rules deployed straight to production without review or testing** = broken logic and false-positive floods reach the SOC directly; a bad rule pages everyone at 3am. Review and CI catch it first.
- **A rule that fires on nothing (or everything)** slipping into production = missing functional testing; CI that checks "fires on malicious, quiet on benign" would have caught it.
- **Detections locked to one SIEM's syntax** = fragile and un-portable; authoring in Sigma and converting in the pipeline keeps the logic reusable and testable.
- **A git repo of reviewed, tested, ATT&CK-mapped detections deploying through a pipeline** = a mature practice; the library is trustworthy, and coverage is visible.

## Pitfalls

- **Editing rules in the console.** The default bad habit — no history, no review, no test, and untracked drift. Version control is the foundation; everything else builds on it.
- **Deploying without review or testing.** Detection logic is as breakable as any code; unreviewed, untested rules cause false-positive floods or silent blind spots. Gate them.
- **Skipping functional testing.** Syntax-valid isn't the same as working; a rule that never fires is assumed coverage you don't have. Test that it fires on real activity.
- **Platform lock-in.** Authoring only in one SIEM's language makes detections un-portable and hard to test; use a portable format where possible.
- **No coverage mapping.** Without ATT&CK metadata the repo can't tell you what you cover; you write duplicate rules and miss gaps.

## References

- Sigma and pySigma / sigma-cli (portable rules + conversion)
- SANS and industry detection-as-code / detection engineering references
- The writing-sigma-rules, testing-detections, mapping-to-attack, and detection-metrics skills
- Git and CI/CD practice (the DevSecOps domain)
