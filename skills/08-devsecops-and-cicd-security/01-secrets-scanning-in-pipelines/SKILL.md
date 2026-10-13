---
format: "v2"
name: "secrets-scanning-in-pipelines"
title: "Secrets Scanning In Pipelines"
title_fr: "Détection de secrets dans les pipelines"
description: "Use when you want to stop credentials from being committed or built into artifacts — wiring secret scanning into pre-commit and CI so leaks are caught before they ship."
description_fr: "À utiliser pour empêcher que des identifiants soient commités ou embarqués dans des artefacts — en intégrant la détection de secrets au pre-commit et à la CI pour intercepter les fuites avant qu'elles ne partent en production."
domain: "08-devsecops-and-cicd-security"
tags: [cybersecurity, engineering, best-practices]
maturity: "stable"
audience: ["backend-engineer", "security-engineer", "coding-agent"]
requires: ["bash", "git"]
updated: "2026-10-13"
---



## Prerequisites
- Target system, dependencies and environment configured.

## Usage
### Purpose

Secrets leak into repos constantly — a key pasted for a quick test, a `.env` committed by accident. Once pushed, the credential is compromised even after you delete it, because history and mirrors keep the copy. This skill covers catching them at two chokepoints (before the commit, and in CI) so they never reach a shared branch.

It's the preventive counterpart to the OSINT github-secret-recon skill: that one finds leaks after the fact, this one stops them happening.

### When to use it

Setting up a new repo, or hardening an existing one that's had a leak scare. High value for low effort — a pre-commit hook plus a CI job blocks the overwhelming majority of accidental secret commits.

### Procedure

1. Add a **pre-commit hook** so the scan runs on the developer's machine before the secret ever leaves it — the earliest and cheapest catch:
   ```yaml
   # .pre-commit-config.yaml
   repos:
     - repo: https://github.com/gitleaks/gitleaks
       rev: v8.30.1
       hooks:
         - id: gitleaks
   ```
   ```
   pre-commit install
   ```
2. Add a **CI job** as the backstop — not every developer will have the hook installed, so the pipeline enforces it for the whole team:
   ```yaml
   # ci step
   - run: gitleaks git --redact --exit-code 1 .
   ```
   The job must fetch the commits it intends to scan. A shallow checkout cannot support a claim that the entire repository history was checked.
3. Scan **full history** once when adopting this on an existing repo — you want to know what's already leaked, not just what's new:
   ```
   gitleaks git --redact --log-opts="--all" .
   ```
4. Tune to cut false positives without going blind. Add a `.gitleaks.toml` allowlist for known test fixtures and example values, but keep it tight — an over-broad allowlist defeats the point.
5. Turn on the platform's native **push protection** where available as a third layer that blocks at the server regardless of local setup.
6. Define the response for a real hit (see below) so a caught secret is actually remediated, not just un-committed.

### Cheatsheet

```bash
gitleaks git --pre-commit --staged --redact .

gitleaks git --redact .

gitleaks git --redact --log-opts="--all" .

gitleaks dir --redact .

gitleaks git --redact --exit-code 1 --report-format sarif --report-path gl.sarif .
```

### Reading the output

- **A finding that may be a live credential** needs prompt owner-side validation and rotation. Do not paste or test its value in an unrelated service.
- **A hit in history but not the current tree** still counts: the secret must be rotated, deletion isn't remediation.
- **Repeated false positives on the same fixture** mean it's time for a scoped allowlist entry — don't let noise train the team to ignore the scanner.
- **A clean CI run** is only as good as the ruleset and the scanned range. `gitleaks git` inspects commits; `gitleaks dir` inspects current files, including files outside Git history. Check which one your pipeline actually runs.

### The response when a real secret is caught

Rotate first — the credential is burned the moment it was committed, so revoke and reissue immediately. Then purge it from history (`git filter-repo`/BFG) and force-push, understanding that mirrors and forks may still hold the old commit (rotation is what actually protects you). Move the secret into a proper secret manager and inject it at runtime so it never lives in the repo again.

### Pitfalls

- **CI-only, no pre-commit.** Catching it in CI means it's already pushed — rotation required. The hook stops it before that.
- **Deleting the commit and calling it done.** Without rotation, the leaked key is still valid to anyone who saw the push.
- **Over-broad allowlists.** One sloppy ignore rule and real secrets slip through silently.
- **No response plan.** A scanner that finds secrets nobody rotates is theatre. Wire the rotation step in.

### References

- [Gitleaks usage and pre-commit hook](https://github.com/gitleaks/gitleaks#usage)
- [Gitleaks release v8.30.1](https://github.com/gitleaks/gitleaks/releases/tag/v8.30.1)
- OWASP DevSecOps Guideline — secrets management
- GitHub/GitLab secret scanning and push protection docs

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.
