---
name: secrets-in-code
domain: 10-secure-code-review
description: Use when reviewing (or sweeping) a codebase for hardcoded credentials, keys, and tokens — finding them, judging severity, and handling the fact that git remembers.
difficulty: beginner
tags: [code-review, secrets, credentials, git]
tools: [ripgrep, gitleaks, trufflehog]
---

## Purpose

A secret in source is a credential you've handed to everyone with read access — every contributor, every fork, every CI log, and anyone who ever clones the repo. This is a shorter skill than most in the domain because the technique is simple; the part people get wrong is what to do *after* you find one.

## When to use it

On every PR (a quick grep pass), and as a periodic full-history sweep. New secrets sneak in through config files, test fixtures, and "temporary" debugging commits that never get cleaned up.

## Finding them

Two moves. Grep for the obvious shapes, and run a dedicated scanner for entropy and known-provider formats.

```bash
# obvious assignments
rg -ni '(password|passwd|secret|api[_-]?key|token|private[_-]?key|aws_secret)\s*[:=]'
# provider key formats
rg -n 'AKIA[0-9A-Z]{16}|ghp_[0-9A-Za-z]{36}|sk-[A-Za-z0-9]{20,}|-----BEGIN.*PRIVATE KEY-----'
# high-entropy strings the eye misses — let a tool do it
gitleaks detect --source . --redact
trufflehog filesystem . --only-verified
```

`--only-verified` on trufflehog matters: it actually tries the credential, so you spend your time on live secrets, not on the hundred example keys in test data.

## Judging what you find

Not every match is an incident. Sort them:

- **Live credential to a real system** (prod DB, cloud key, signing key) → treat as compromised the moment it touched the repo. Rotate first, clean up second.
- **Test/example/placeholder value** (`password = "changeme"`, obvious dummy keys) → not a secret, but worth a comment so the next scanner run and the next reviewer don't re-flag it.
- **Secret in config that's *supposed* to be injected** but got a real default value committed → common and easy to miss; the placeholder pattern is right, the value is wrong.
- **Something in git history but removed from HEAD** → still exposed. Removal from the current file does nothing; the object is still in the history.

## The part people get wrong

Deleting the line and committing does **not** remove the secret. It's in every prior commit, in every clone, and probably in a CI log already. So the order is:

1. **Rotate the credential.** Assume it's known. This is the only step that actually restores security; everything else is cleanup.
2. Remove it from the code and move it to a secrets manager / env injection.
3. Optionally scrub history (`git filter-repo`, BFG) — but only *after* rotation, and know that anyone who cloned already has it. Rewriting history on a shared repo is disruptive; rotation is what protects you regardless.

I'd rather see a PR that rotates and leaves the history dirty than one that rewrites history and forgets to rotate. Rotation is the fix. History scrubbing is hygiene.

## Preventing the next one

- A pre-commit hook running `gitleaks` or `trufflehog` catches most before they land — recommend it if the repo has none.
- Config via environment/secret store, with committed files holding only placeholders.
- A `.gitignore` that actually covers `.env`, keystores, and credential files — check it exists and is right.

## Pitfalls

- **Deleting without rotating.** The single most common mistake, and it leaves you fully exposed while feeling fixed.
- **Only scanning HEAD.** The interesting secrets are usually in history.
- **Alert fatigue from unverified hits.** Lean on verified/entropy scanners so test fixtures don't bury the real one.
- **Assuming private repo = safe.** CI logs, forks, contractors, and future open-sourcing all leak it.

## References

- OWASP Secrets Management Cheat Sheet
- CWE-798 (Hardcoded Credentials), CWE-540 (Info in Source Code)
- gitleaks, trufflehog, git-filter-repo / BFG docs
