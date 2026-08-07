---
name: sast-integration
domain: 08-devsecops-and-cicd-security
description: Use when wiring static application security testing into the pipeline — catching code vulnerabilities on every build without drowning developers in false positives.
difficulty: intermediate
tags: [devsecops, sast, cicd, static-analysis, shift-left]
tools: [semgrep, sonarqube, codeql]
---

## Purpose

Static Application Security Testing scans source code for vulnerability patterns automatically — injection flaws, hardcoded secrets, unsafe deserialization — catching them at commit time instead of in production. But SAST integrated badly is worse than none: false-positive floods that developers learn to ignore, or a gate so slow it gets bypassed. This skill covers integrating SAST into CI/CD so it catches real bugs and developers actually act on the results — shifting security left effectively.

## When to use it

Building security into the development pipeline, and it's usually one of the first "shift-left" controls teams add. The value is entirely in the integration quality — the tool matters less than whether developers trust and act on its output.

## Procedure

1. **Run SAST in the pipeline on every change.** Wire the scanner (Semgrep, CodeQL, SonarQube) into CI so it runs on pull requests and commits — catching issues before merge, when they're cheapest to fix. Static analysis on every change is the shift-left goal.
2. **Tune to cut false positives — the make-or-break factor.** SAST tools generate false positives, and a noisy scanner trains developers to ignore all of it, including the real findings. Tune rules to your codebase, suppress confirmed false positives with reasons, and start with high-confidence rules. Developer trust in the output is what determines whether SAST works.
3. **Give fast, actionable feedback in the developer's workflow.** Results should appear on the pull request (inline, at the vulnerable line) with a clear explanation and fix guidance — not in a separate portal nobody checks. Fast, in-context feedback is what gets issues fixed; slow or out-of-band results get ignored.
4. **Gate carefully — fail builds on high-confidence, high-severity findings only.** Blocking merges on every finding (including false positives) causes revolt and bypassing; blocking on nothing means findings pile up unfixed. Fail the build only on high-confidence, high-severity issues; report the rest without blocking. Calibrate the gate to keep developer trust.
5. **Manage the backlog of existing findings.** Turning SAST on for an existing codebase surfaces thousands of pre-existing issues; don't block all development on them. Baseline the existing findings and gate only on *new* ones introduced by a change (differential scanning), so the pipeline stays usable while debt gets worked down.
6. **Prioritise findings by real risk** — reachable, exploitable issues over theoretical ones (the code-review and vuln-mgmt prioritisation carries over). Not every SAST finding matters equally.
7. **Make it maintainable and measured.** SAST config is code — version it, review rule changes, and track metrics (findings fixed, false-positive rate) to keep it healthy and prove value.

## Cheatsheet

```
SAST = scan source for vuln patterns on every build (injection, secrets, unsafe deser)
  shift-left: catch at commit, not production. BUT bad integration < none.

integrate well (the value is here, not the tool)
  RUN in CI on PRs/commits (catch before merge)
  TUNE false positives (make-or-break): noisy -> devs ignore ALL of it incl. real findings
    -> tune to codebase, suppress-with-reason, high-confidence rules first
  FAST + IN-WORKFLOW feedback: on the PR, at the line, with fix guidance (not a portal)
  GATE carefully: fail build on HIGH-confidence + HIGH-severity ONLY
    (block everything -> revolt/bypass ; block nothing -> pile up unfixed)
  EXISTING codebase: baseline old findings, gate on NEW ones (differential) — keep pipeline usable
  PRIORITISE by real risk (reachable/exploitable > theoretical)
  config as CODE + metrics (fixed, FP rate)
```

## Reading the integration

- **A noisy SAST scanner** = the top failure; false-positive floods train developers to ignore everything, including real vulnerabilities. Tuning for developer trust is what makes SAST work — an untuned scanner is worse than none.
- **Findings in a separate portal nobody checks** = they don't get fixed; results must appear in the developer's workflow (on the PR, at the line) with fix guidance. In-context, fast feedback is what drives remediation.
- **A gate that fails builds on every finding** = developer revolt and bypassing; blocking on false positives destroys trust in the whole control. Gate only high-confidence, high-severity issues.
- **Turning SAST on and blocking all development on thousands of pre-existing findings** = an unusable pipeline; baseline existing debt and gate on new findings so development continues while debt is worked down.
- **Findings prioritised by real risk** (reachable, exploitable) = developers fix what matters; treating every finding as equal buries the important ones.
- **Tuned, in-workflow, differentially-gated SAST with trusted output** = shift-left working; real bugs caught at commit and actually fixed.

## Pitfalls

- **Ignoring false positives.** A noisy scanner trains developers to dismiss all findings; the real vulnerabilities get ignored with the noise. Tuning for trust is the single most important integration factor.
- **Out-of-band results.** Findings in a portal nobody checks don't get fixed; deliver them in the PR at the vulnerable line with guidance.
- **Over-gating.** Failing builds on every finding (including false positives) causes revolt and bypassing; gate on high-confidence, high-severity only.
- **Blocking on existing debt.** Turning SAST on and gating all development on pre-existing findings makes the pipeline unusable; baseline old findings, gate on new.
- **Treating all findings equally.** Prioritise by reachability and exploitability; not every pattern match is a real risk.

## References

- Semgrep, CodeQL, and SonarQube documentation
- OWASP DevSecOps guideline and the secure-code-review domain (finding validation)
- The dependency-scanning, dast-in-cicd, and policy-as-code skills
- OWASP SAST guidance
