---
format: "v2"
name: "github-secret-recon"
title: "Github Secret Recon"
title_fr: "Recherche de secrets sur GitHub"
description: "Use when checking whether an organisation has leaked API keys, credentials, or internal infrastructure details in public git repositories."
description_fr: "À utiliser pour vérifier si une organisation a laissé fuiter des clés API, des identifiants ou des détails d'infrastructure interne dans des dépôts git publics."
domain: "01-osint-and-reconnaissance"
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

Developers commit secrets. Then they force-push a fix and think it's gone — but git keeps history, and public mirrors keep copies. This skill covers finding an org's leaked credentials and infra details across public repos, so you find them before someone hostile does.

### When to use it

External recon on an org that ships code publicly, or a self-audit of your own GitHub presence. High signal, because a live cloud key in a repo is often a straight line to production.

Search only public data, and never *use* a credential you find during unauthorised recon — finding it is recon, using it is intrusion. Report it.

### Procedure

1. List the org's public repos, and don't stop at the main org — check personal accounts of known employees and forks, where leaks hide:
   ```
   gh repo list <org> --limit 200 --json name,url
   ```
2. Scan a repo's **full history**, not just the current tree. The secret is usually in an old commit that a later commit "removed":
   ```
   trufflehog github --repo=https://github.com/org/repo
   ```
3. Run a second scanner over a local clone — different tools catch different patterns:
   ```
   git clone --mirror https://github.com/org/repo repo.git
   gitleaks detect --source repo.git
   ```
4. Beyond credentials, read for infrastructure leaks: internal hostnames, S3 bucket names, CI config, `.env.example` files that reveal the shape of the real `.env`.
5. Verify a hit is real and live before you raise it — scanners flag test/placeholder values too. Confirm the format, and if it's your own authorised audit, check whether the key still authenticates.

### Cheatsheet

```bash
trufflehog github --org=<org> --only-verified

trufflehog github --repo=https://github.com/org/repo

git clone --mirror <url> r.git && gitleaks detect --source r.git -v

gh api -X GET search/code -f q="org:<org> password"
```

`--only-verified` in trufflehog actively checks whether a found key still works — huge for cutting false positives, but it does make a request with the credential, so only use it on your own authorised audit.

### Reading the output

- A **verified live key** (AWS, Stripe, a signing secret) is critical — treat it as an active exposure, not a hypothetical.
- **A hit in old history but not the current file** is still a real leak; the credential must be rotated, deleting the commit isn't enough.
- **Placeholder-looking values** (`AKIAEXAMPLE`, `changeme`) are usually noise — confirm before reporting.
- **Internal hostnames and bucket names** aren't secrets but widen the attack surface for other domains here.

### The fix (for your own leaks)

Rotate first, always — the secret is compromised the moment it's public, and history rewriting doesn't un-leak it. Then purge it from history (`git filter-repo` or BFG) and force-push. Prevent the next one with a pre-commit secret scanner and a server-side push protection rule, plus a `.gitignore` that actually covers `.env` and key files. Move real secrets into a secret manager, not the repo.

### Pitfalls

- **Scanning current tree only.** Misses the majority of leaks, which live in history.
- **Ignoring employee personal repos.** A lot of leaks are in someone's side project, not the org account.
- **Treating history deletion as remediation.** Rotate the credential — mirrors and forks already have the old commit.
- **Alert fatigue.** Without verification you drown in test values. Verify, then prioritise the live ones.

### References

- OWASP WSTG-INFO — Review Webpage Content / source for leaks
- trufflehog and gitleaks documentation
- GitHub docs — Secret scanning and push protection

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.