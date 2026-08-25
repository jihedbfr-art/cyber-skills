---
format: "v2"
name: "reviewing-a-pr-for-security"
title: "Reviewing A Pr For Security"
title_fr: "Revue de sécurité d'une pull request"
description: "Use as a repeatable, time-boxed checklist for the security pass on an everyday pull request — where to look first, what to wave through, when to block."
description_fr: "À utiliser comme checklist reproductible et limitée dans le temps pour la passe de sécurité d'une pull request ordinaire — où regarder en premier, quoi laisser passer, quand bloquer."
domain: "10-secure-code-review"
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

The other skills in this domain go deep on one bug class each. This one is the glue: how to run a security pass on a normal PR in the ten or fifteen minutes you actually have, without either rubber-stamping it or turning every review into a full audit. It's a workflow, so it reads as a checklist rather than a bug write-up.

### When to use it

Every PR you review, as a distinct pass after (or alongside) the functional review. The goal is a consistent minimum bar, not perfection — most PRs need five minutes of the right questions, and a few need you to stop and dig.

### Start by sizing the risk

Not every PR deserves equal scrutiny. Before reading line by line, ask what the change *touches*:

- **High attention:** authentication/authorization, anything handling user input, queries, file or path handling, crypto, payments, deserialisation, dependency or config changes, new endpoints, anything near secrets.
- **Low attention:** copy changes, styling, internal refactors with no boundary crossing, test-only changes, docs.

Spend your budget where the blast radius is. A CSS PR and a change to the login flow are not the same review.

### The pass — what to check

Walk the diff with these questions. Each maps to a deeper skill in this domain if it lights up.

1. **New input, new sink?** Does the PR introduce a source (request data, upload, external response) or a sink (query, exec, HTML, path, deserialiser)? If both, taint-track the path. → `taint-tracking-by-hand`, `injection-patterns`
2. **Access control on new/changed endpoints?** Every new by-id fetch needs an ownership/tenant check; every privileged action needs a server-side role check. Identity must come from the session/token, not the payload. → `auth-and-authz-review`
3. **Secrets?** Any key, token, password, or connection string in the diff — including test fixtures and config. → `secrets-in-code`
4. **Crypto or randomness?** New hashing, encryption, token/id generation. Watch for fast hashes on passwords and `Math.random()` for anything security-relevant. → `crypto-misuse-review`
5. **Deserialisation of untrusted data?** Native serialisers, polymorphic typing switched on. → `deserialization-review`
6. **Concurrency on limited resources?** Check-then-act on balances, quotas, one-time tokens; shared mutable state. → `race-conditions`
7. **Errors and logs.** Internal detail leaking to the client; swallowed security failures; secrets or raw user input in logs. → `error-handling-and-logging`
8. **Dependencies and config.** New/updated deps (scan them), and any config or framework default that shipped unsafe. → `dependency-and-config-review`

### A fast triage order

If you only have a few minutes: check secrets and access control first (highest-value, fastest to spot), then anything with untrusted input reaching a sink. Crypto, deserialisation, and concurrency are rarer per-PR but higher-severity when present — slow down the moment you see them.

### Writing the review comment

- Be specific and reproducible: name the file, line, the source→sink path, and why it's exploitable — not "this looks insecure." A reviewer's finding is only as good as the maintainer's ability to act on it. (The public-writing habits in the repo's contribution guide apply here too: say it once, plainly.)
- Separate **blocking** from **suggestion**. Block on exploitable, reachable issues (injection, broken authz, live secret, RCE-class deserialisation). Suggest on hardening and defence-in-depth. Muddling the two gets your blocking comments ignored.
- Offer the fix, not just the flaw, when it's a known pattern (parameterise this, bind identity from the token, move this to a secrets store).
- When you're unsure, say so and ask — "is this endpoint reachable unauthenticated?" is a legitimate review comment. Guessing confidently is worse than asking.

### When to block

Block when there's a reachable path to: injection, broken access control / IDOR, a live secret, unsafe deserialisation of untrusted data, disabled TLS verification or CSRF in prod, or an exploitable dependency in the code path. Everything else is a conversation, not a gate. Don't block a PR to make it *perfect* — block it to keep an exploitable bug out, and file the rest as follow-ups.

### Pitfalls

- **Uniform scrutiny.** Reading a docs PR as hard as an auth PR wastes the budget you needed for the auth PR.
- **Vague findings.** "Feels insecure" gets dismissed; a concrete path gets fixed.
- **Blocking on style-of-security.** Reserve the block for exploitable; over-blocking trains people to route around you.
- **Reviewing only the diff.** A diff can be safe in isolation and break an invariant elsewhere — for auth and concurrency especially, glance at the surrounding code the change relies on.

### References

- OWASP Code Review Guide; OWASP Top 10 as the risk backbone
- This domain's other nine skills — this checklist is their index
- The repo's contribution guide for review-comment tone

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.