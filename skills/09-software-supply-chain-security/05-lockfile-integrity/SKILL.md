---
format: "v2"
name: "lockfile-integrity"
title: "Lockfile Integrity"
title_fr: "Intégrité des fichiers de verrouillage"
description: "Use when ensuring reproducible, verified dependencies — using lockfiles with integrity hashes so you install exactly the packages you vetted, and nothing gets swapped."
description_fr: "À utiliser pour garantir des dépendances reproductibles et vérifiées — s'appuyer sur des lockfiles à hachages d'intégrité afin d'installer exactement les paquets validés, sans qu'aucun ne soit substitué."
domain: "09-software-supply-chain-security"
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

A lockfile records the exact versions and integrity hashes of every dependency (including transitive ones) your project resolved. It's a foundational, low-effort supply-chain control: it makes builds reproducible (everyone installs the same thing) and verifiable (the integrity hash ensures a package wasn't swapped or tampered with between resolution and install). This skill covers using lockfiles for integrity, the simple discipline that underpins dependency-confusion, typosquat, and tampering defences.

### When to use it

Every project with dependencies (nearly all). It's basic and often already present but under-used or not enforced — a lockfile that isn't committed, verified, or trusted provides little of its value. It's the substrate the other supply-chain controls build on.

### Procedure

1. **Commit the lockfile.** The lockfile (`package-lock.json`, `poetry.lock`, `Cargo.lock`, `pip` with hashes, etc.) must be committed to version control so everyone — developers and CI — resolves to the exact same dependency tree. An uncommitted lockfile means builds can resolve differently, defeating reproducibility.
2. **Install from the lockfile, strictly.** Use the install mode that installs *exactly* what the lockfile specifies and *fails* if the lockfile is out of sync, rather than silently re-resolving. `npm ci` (not `npm install`), `pip install --require-hashes`, `poetry install` — these enforce the locked state:
   ```
   npm ci                        # installs exactly the lockfile, fails on mismatch
   pip install --require-hashes  # verifies integrity hashes
   ```
   `npm install` can silently update the lockfile; `npm ci` enforces it.
3. **Verify integrity hashes.** The lockfile's integrity hashes are the tamper-protection: on install, the package manager checks the downloaded package's hash against the lockfile, rejecting a package that doesn't match (swapped, tampered, or a registry substitution). Ensure hash verification is on (some ecosystems require explicit config, like pip's `--require-hashes`).
4. **Review lockfile changes.** A change to the lockfile means dependencies changed; review lockfile diffs in pull requests. An unexpected dependency appearing, or a hash changing without a version change, is a red flag (possible tampering or a suspicious update).
5. **Regenerate deliberately.** Update the lockfile intentionally (when adding/updating dependencies), not accidentally; a lockfile that drifts silently loses its guarantee. Automated dependency-update tools (Dependabot) regenerate it cleanly with review.
6. **Combine with the other controls.** Lockfile integrity ensures you install what you resolved; pair with dependency scanning (are the locked versions vulnerable?), typosquat/confusion defences (is the resolved package the right one?), and signing.

### Cheatsheet

```
lockfile = exact versions + INTEGRITY HASHES of every dependency (incl. transitive)
  -> reproducible (same install everywhere) + verifiable (hash = not swapped/tampered)
  foundational, low-effort. often present but under-used/unenforced.

do
  COMMIT the lockfile (uncommitted -> builds resolve differently)
  INSTALL STRICTLY from it, fail on mismatch (NOT silent re-resolve)
    npm ci (not npm install) | pip install --require-hashes | poetry install
  VERIFY integrity hashes (tamper protection — reject package whose hash != lockfile)
    (some ecosystems need explicit config: pip --require-hashes)
  REVIEW lockfile diffs in PRs (unexpected dep / hash change w/o version change = red flag)
  REGENERATE deliberately (adding/updating deps) — silent drift loses the guarantee
  COMBINE with dependency scanning + typosquat/confusion defences + signing
```

### Reading the practice

- **An uncommitted lockfile** = builds can resolve to different dependency trees; reproducibility and the integrity guarantee are lost. Commit it — the foundational step.
- **Using `npm install` (or loose install) in CI** = the lockfile can be silently updated and different packages installed; use `npm ci` (or the strict equivalent) that installs exactly the lockfile and fails on mismatch. The difference between a lockfile that's enforced and one that's decorative.
- **Integrity hash verification off** = the tamper protection is disabled; a swapped or tampered package installs unnoticed. Ensure hash checking is on (pip needs `--require-hashes`).
- **A hash changing without a version change in a lockfile diff** = a red flag — the package content changed for the same version, possibly tampering or a compromised republish. Review lockfile diffs.
- **A silently-drifting lockfile** = the guarantee erodes; regenerate deliberately with review, not accidentally.
- **Committed, strictly-installed, hash-verified, reviewed lockfiles** = reproducible, tamper-resistant dependency installs — the substrate the other supply-chain controls need.

### Pitfalls

- **Not committing the lockfile.** Without it committed, everyone resolves differently and the integrity guarantee is gone. Commit it.
- **Loose install commands.** `npm install` silently updates the lockfile; use `npm ci` (and strict equivalents) that enforce the locked state and fail on mismatch. Otherwise the lockfile is decorative.
- **Hash verification disabled.** The integrity hashes are the tamper protection; without verification (e.g. pip without `--require-hashes`), a swapped package installs unnoticed.
- **Ignoring lockfile diffs.** Dependency and hash changes carry supply-chain risk; a hash change without a version change is suspicious. Review lockfile changes in PRs.
- **Silent regeneration.** A lockfile that drifts accidentally loses its guarantee; update it deliberately with review.

### References

- npm (`npm ci`, package-lock), pip (`--require-hashes`), Poetry, Cargo lockfile documentation
- The dependency-confusion, typosquat-detection, and vulnerable-dependency-triage skills
- The devsecops dependency-scanning skill
- OpenSSF supply-chain security guidance

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.