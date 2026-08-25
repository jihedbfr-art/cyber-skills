---
format: "v2"
name: "dast-in-cicd"
title: "Dast In Cicd"
title_fr: "DAST dans la CI/CD"
description: "Use when running dynamic application security testing in the pipeline — scanning a running application for vulnerabilities automatically, and where DAST fits versus SAST."
description_fr: "À utiliser pour exécuter des tests dynamiques de sécurité applicative (DAST) dans le pipeline — en scannant automatiquement une application en cours d'exécution à la recherche de vulnérabilités, et pour situer le DAST par rapport au SAST."
domain: "08-devsecops-and-cicd-security"
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

Dynamic Application Security Testing scans a *running* application from the outside — sending requests and observing responses — to find vulnerabilities that only appear at runtime: authentication flaws, misconfigurations, injection that manifests in behaviour. Where SAST reads the code, DAST exercises the deployed app. This skill covers running DAST in CI/CD against ephemeral test environments, and understanding where it complements SAST rather than duplicating it.

### When to use it

Adding runtime security testing to the pipeline, complementing the static analysis (SAST) and dependency scanning. It's most valuable for web applications and APIs, and it catches a different class of issue than SAST — the two together give fuller coverage.

### DAST vs SAST (where each fits)

- **SAST** reads source code (white-box), finds code-level flaws (injection patterns, unsafe functions), runs early (on commit, no running app needed), but produces false positives and can't see runtime/config issues.
- **DAST** exercises the running app (black-box), finds runtime and configuration flaws (auth bypasses, missing headers, server misconfig, injection that actually works), needs a deployed environment, is slower, but its findings are real (it confirmed the behaviour). They're complementary — neither replaces the other.

### Procedure

1. **Deploy an ephemeral test environment for DAST to scan.** DAST needs a running application; spin up a temporary environment (per-PR or per-pipeline) that mirrors production, run DAST against it, and tear it down. Scanning ephemeral environments keeps DAST in the pipeline without a permanent target.
2. **Run an automated scanner** (OWASP ZAP is the standard for CI, Nuclei for template-based checks) against the deployed app. Configure it to crawl the app and test for the common vulnerability classes.
3. **Handle authentication.** Much of an app is behind login; configure DAST with credentials/session handling so it can scan authenticated areas, or it only tests the public surface. Authenticated scanning is where most of the coverage is.
4. **Scope and time-box the scan.** Full DAST scans are slow (too slow for every commit); use a fast baseline scan in the pipeline (quick, common checks) and schedule deeper scans periodically. Balance coverage against pipeline speed.
5. **Tune and triage results.** DAST has fewer false positives than SAST (it confirmed the behaviour), but still needs triage — some findings are informational or low-impact. Prioritise the real, exploitable ones (the vuln-mgmt prioritisation).
6. **Gate appropriately.** Fail the pipeline on high-confidence, high-severity DAST findings (a confirmed injection, an auth bypass); report the rest. As with SAST, over-gating causes bypassing.
7. **Combine with SAST and dependency scanning** for layered coverage — DAST catches what static analysis can't (runtime/config), and vice versa. Don't treat one as sufficient.

### Cheatsheet

```
DAST = scan the RUNNING app from outside (requests + responses) -> runtime/config flaws
  SAST reads code (early, white-box, FP-prone, misses runtime)
  DAST exercises app (black-box, needs deployed env, slower, findings are REAL/confirmed)
  -> COMPLEMENTARY, neither replaces the other

integrate
  EPHEMERAL test env per PR/pipeline (deploy -> scan -> tear down) — running target needed
  scanner: OWASP ZAP (CI standard) / Nuclei (templates) — crawl + test common classes
  AUTHENTICATION: configure creds/session -> scan authenticated areas (most of the coverage)
  SCOPE/time-box: fast BASELINE scan in pipeline + deeper scans SCHEDULED (full = too slow per commit)
  TUNE/triage (fewer FPs than SAST but still needed) ; prioritise exploitable
  GATE on high-confidence high-severity (auth bypass, confirmed injection) ; report rest
  COMBINE with SAST + dependency scanning (layered coverage)
```

### Reading DAST results

- **A confirmed injection or auth bypass from DAST** = a real, exploitable finding (DAST observed the behaviour, not just a pattern); high-confidence and gate-worthy. This is DAST's strength — its findings are confirmed, unlike many SAST hits.
- **Missing/weak security headers, server misconfigurations** = runtime/config issues SAST can't see; DAST catches these because it exercises the deployed app. Complementary coverage.
- **DAST only testing the public surface** = a coverage gap; most of an app is behind login. Configure authenticated scanning or you miss the majority of the attack surface.
- **A full scan too slow for every commit** = expected; use a fast baseline in the pipeline and schedule deeper scans. Over-slow gates get bypassed.
- **Treating DAST as a replacement for SAST** (or vice versa) = a coverage gap; they find different classes of issue. Use both.
- **Ephemeral-environment DAST with authenticated scanning, tuned and layered with SAST** = full pipeline security coverage, static and dynamic.

### Pitfalls

- **Unauthenticated scanning only.** Most of an app is behind login; without authenticated DAST you only test the public surface and miss the bulk of the vulnerabilities. Configure session handling.
- **Full scans on every commit.** Too slow; the gate gets bypassed. Use a fast baseline in the pipeline and schedule deeper scans.
- **Treating DAST or SAST as sufficient alone.** They catch different classes (runtime/config vs code-level); one without the other leaves gaps. Layer them.
- **No ephemeral environment.** DAST needs a running target; without a per-pipeline deployable environment, it can't run in CI. Automate the environment.
- **Over-gating.** Blocking on every DAST finding (including informational ones) causes bypassing; gate on high-confidence, high-severity.

### References

- OWASP ZAP (and its CI/automation modes), Nuclei documentation
- The sast-integration and dependency-scanning skills (complementary controls)
- The web-application-security and API-security domains (what DAST tests for)
- OWASP DevSecOps and DAST guidance

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.