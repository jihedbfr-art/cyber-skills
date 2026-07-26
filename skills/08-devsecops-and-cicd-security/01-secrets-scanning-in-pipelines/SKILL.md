---
name: secrets-scanning-in-pipelines
domain: 08-devsecops-and-cicd-security
description: Use when you want to stop credentials from being committed or built into artifacts — wiring secret scanning into pre-commit and CI so leaks are caught before they ship.
difficulty: beginner
tags: [devsecops, cicd, secrets, pipeline, prevention]
tools: [gitleaks, trufflehog, pre-commit]
---

## Purpose

Secrets leak into repos constantly — a key pasted for a quick test, a `.env` committed by accident. Once pushed, the credential is compromised even after you delete it, because history and mirrors keep the copy. This skill covers catching them at two chokepoints (before the commit, and in CI) so they never reach a shared branch.

It's the preventive counterpart to the OSINT github-secret-recon skill: that one finds leaks after the fact, this one stops them happening.

## When to use it

Setting up a new repo, or hardening an existing one that's had a leak scare. High value for low effort — a pre-commit hook plus a CI job blocks the overwhelming majority of accidental secret commits.

## Procedure

1. Add a **pre-commit hook** so the scan runs on the developer's machine before the secret ever leaves it — the earliest and cheapest catch:
   ```yaml
   # .pre-commit-config.yaml
   repos:
     - repo: https://github.com/gitleaks/gitleaks
       rev: v8.18.0
       hooks:
         - id: gitleaks
   ```
   ```
   pre-commit install
   ```
2. Add a **CI job** as the backstop — not every developer will have the hook installed, so the pipeline enforces it for the whole team:
   ```yaml
   # ci step
   - run: gitleaks detect --source . --redact --exit-code 1
   ```
3. Scan **full history** once when adopting this on an existing repo — you want to know what's already leaked, not just what's new:
   ```
   gitleaks detect --source . --log-opts="--all"
   ```
4. Tune to cut false positives without going blind. Add a `.gitleaks.toml` allowlist for known test fixtures and example values, but keep it tight — an over-broad allowlist defeats the point.
5. Turn on the platform's native **push protection** (GitHub/GitLab secret scanning) as a third layer that blocks at the server regardless of local setup.
6. Define the response for a real hit (see below) so a caught secret is actually remediated, not just un-committed.

## Cheatsheet

```bash
# local, staged changes (pre-commit does this automatically)
gitleaks protect --staged --redact

# whole working tree
gitleaks detect --source . --redact

# full history (adoption audit)
gitleaks detect --source . --log-opts="--all"

# CI: fail the build on any finding
gitleaks detect --source . --exit-code 1 --report-format sarif --report-path gl.sarif

# second scanner for coverage
trufflehog git file://. --only-verified
```

## Reading the output

- **A verified live credential** (trufflehog's `--only-verified` confirms it still works) is the top priority — treat as an active exposure.
- **A hit in history but not the current tree** still counts: the secret must be rotated, deletion isn't remediation.
- **Repeated false positives on the same fixture** mean it's time for a scoped allowlist entry — don't let noise train the team to ignore the scanner.
- **A clean CI run** is only as good as the ruleset; periodically confirm the scanner still detects a planted test secret.

## The response when a real secret is caught

Rotate first — the credential is burned the moment it was committed, so revoke and reissue immediately. Then purge it from history (`git filter-repo`/BFG) and force-push, understanding that mirrors and forks may still hold the old commit (rotation is what actually protects you). Move the secret into a proper secret manager and inject it at runtime so it never lives in the repo again.

## Pitfalls

- **CI-only, no pre-commit.** Catching it in CI means it's already pushed — rotation required. The hook stops it before that.
- **Deleting the commit and calling it done.** Without rotation, the leaked key is still valid to anyone who saw the push.
- **Over-broad allowlists.** One sloppy ignore rule and real secrets slip through silently.
- **No response plan.** A scanner that finds secrets nobody rotates is theatre. Wire the rotation step in.

## References

- gitleaks and trufflehog documentation
- OWASP DevSecOps Guideline — secrets management
- GitHub/GitLab secret scanning and push protection docs
