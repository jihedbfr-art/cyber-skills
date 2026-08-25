---
format: "v2"
name: "crypto-agility"
title: "Crypto Agility"
title_fr: "Agilité cryptographique"
description: "Use when designing systems so cryptographic algorithms and keys can be changed later without a rewrite — because every algorithm eventually needs replacing."
description_fr: "À utiliser pour concevoir des systèmes permettant de changer d'algorithmes et de clés cryptographiques plus tard sans réécriture — car tout algorithme finit par devoir être remplacé."
domain: "14-cryptography-and-pki"
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

Every cryptographic algorithm has a shelf life — MD5, SHA-1, RSA-1024, DES were all "fine" once. The systems that handled their retirement gracefully were the ones designed to swap algorithms without a rewrite; the ones that hardcoded a single algorithm faced painful, risky migrations. Crypto-agility is designing so that changing algorithms and keys is a configuration/rollout task, not a re-architecture. This skill covers building that flexibility in from the start.

### When to use it

Designing any system that uses cryptography and will live for years (most of them), or reviewing a system for how hard an algorithm migration would be. It's forward-looking — the payoff comes when an algorithm you depend on gets deprecated or broken, which *will* happen.

### Procedure

1. **Don't hardcode the algorithm — make it a parameter.** Design so the algorithm in use is a configurable choice, not baked into the logic. This is the core of agility: you should be able to change from algorithm A to B without rewriting the code that uses it.
2. **Store the algorithm/version with the data.** Tag encrypted data, hashes, and tokens with which algorithm and key produced them (a version identifier, `kid`, or algorithm field). Then old data remains decryptable/verifiable with the old algorithm while new data uses the new one — the key to migrating without a flag day:
   ```
   # store: {version: 2, alg: "argon2id", ...} alongside the value
   # verify/decrypt using the stored version; write with the current version
   ```
3. **Support side-by-side old and new during migration.** Real migrations aren't instant — you need to read old-format data while writing new-format. Design for a transition period where both are valid, and migrate opportunistically (e.g. re-hash a password on next login, re-encrypt data as it's touched or in a background job).
4. **Design key rotation in from the start** (key-management skill) — algorithm changes and key changes use the same versioning machinery; a system built for key rotation is most of the way to algorithm agility.
5. **Abstract crypto behind an interface** so the rest of the code calls "encrypt/verify" without knowing the algorithm, and the implementation can be swapped centrally. Avoid scattering algorithm-specific calls throughout the codebase.
6. **Keep a migration plan ready** — know how you'd move off each algorithm you depend on, so when a deprecation lands (or PQC arrives — see that skill), it's an execution, not a scramble.

### Cheatsheet

```
the reality: every algorithm eventually needs replacing (MD5/SHA1/RSA-1024/DES...)
crypto-agility = swap algorithm/key WITHOUT a rewrite

design principles
  [ ] algorithm is a PARAMETER, not hardcoded
  [ ] store algorithm/version WITH the data (version id / kid / alg field)
        -> old data verifiable with old alg, new data uses new alg
  [ ] support OLD + NEW side-by-side during migration (no flag day)
        migrate opportunistically: re-hash on login, re-encrypt on touch / background
  [ ] design key rotation in from day one (same machinery as alg change)
  [ ] abstract crypto behind an interface (swap centrally, not scattered calls)
  [ ] keep a migration plan ready for each algorithm you depend on

payoff: when an algorithm is deprecated/broken, it's execution, not re-architecture.
```

### Reading a design

- **A hardcoded algorithm with no version stored** = a painful, risky migration when that algorithm is deprecated — you can't tell old data from new and can't run both. The anti-pattern crypto-agility prevents.
- **Encrypted data / hashes with no algorithm/version tag** = you can't migrate incrementally; changing the algorithm means you can't decrypt/verify existing data. Store the version with the data.
- **Crypto calls scattered algorithm-specifically across the codebase** = a change touches many places and risks inconsistency. Abstract behind an interface.
- **No support for reading old while writing new** = any migration requires a flag day (re-encrypt everything at once), which is risky and often impractical at scale. Support side-by-side.
- **A system that already does key rotation with versioning** = most of the agility work is done; algorithm change uses the same mechanism.
- **Parameterised algorithm, version-tagged data, side-by-side migration, abstracted interface** = agile; algorithm retirement is manageable.

### The fix / best practice

- **Parameterise the algorithm** and store the algorithm/version alongside every encrypted value, hash, and token, so old and new coexist.
- **Support reading old-format while writing new-format**, and migrate opportunistically (on-access re-encryption/re-hashing, background jobs) rather than in a flag day.
- **Abstract cryptography behind an interface** so implementations swap centrally.
- **Design key rotation in from the start** — the same versioning machinery serves algorithm change.
- **Maintain migration plans** for the algorithms you depend on, so deprecations (and the coming PQC transition) are executed calmly.

### Pitfalls

- **Hardcoding a single algorithm.** It works until the algorithm is broken/deprecated, then the migration is a rewrite. Parameterise from the start.
- **Not versioning stored data.** Without an algorithm/version tag, you can't distinguish or migrate old data incrementally — a flag-day migration is the only option, and often infeasible.
- **Scattered algorithm-specific calls.** A change becomes error-prone surgery across the codebase; abstract it.
- **Assuming an algorithm is permanent.** None are; designing as if the current choice is forever guarantees a painful future migration.
- **Ignoring agility until forced.** Building it in early is cheap; retrofitting it under the pressure of a deprecation is expensive and risky.

### References

- NIST guidance on cryptographic agility and algorithm transitions (SP 800-131A)
- The key-management, password-hashing, and post-quantum-readiness skills (all rely on agility)
- OWASP Cryptographic Storage Cheat Sheet (versioning/migration)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.