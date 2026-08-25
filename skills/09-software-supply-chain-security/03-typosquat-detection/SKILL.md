---
format: "v2"
name: "typosquat-detection"
title: "Typosquat Detection"
title_fr: "Détection de typosquatting"
description: "Use when defending against typosquatted and malicious lookalike packages — catching the malicious package with a name close to a popular one before a typo pulls it into your build."
description_fr: "À utiliser pour se défendre contre les paquets malveillants imitant des paquets populaires (typosquatting) — repérer le paquet au nom trompeur avant qu'une faute de frappe ne l'introduise dans le build."
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

Attackers publish malicious packages with names deliberately close to popular ones — `reqeusts` for `requests`, `python3-dateutil` for `python-dateutil` — betting on a typo or a copied-wrong name pulling their code into someone's build. Typosquatting (and related name-confusion attacks) are a constant, low-effort supply-chain threat. This skill covers detecting and preventing typosquatted and malicious lookalike packages entering your dependencies.

### When to use it

Protecting the dependency intake for any project using public registries, and vetting new dependencies before adoption. It complements dependency confusion (same-name attacks) with the similar-name variant, and pairs with malicious-package response.

### Procedure

1. **Understand the name-confusion variants** an attacker uses:
   - **Typosquatting** — a name one keystroke off a popular package (`reqeusts`, `loadsh`).
   - **Combosquatting** — adding a plausible word (`requests-oauth`, `python-requests`).
   - **Namespace/manager confusion** — a name from a different ecosystem, or a slightly different naming convention.
   The bet is a typo, a mis-copied name, or a plausible-but-wrong assumption.
2. **Vet packages before adding them.** When adopting a new dependency, verify you have the *exact* correct name and it's the genuine, popular package (check download counts, maintainer, repository, age). A brand-new package with a name close to a popular one and few downloads is a red flag.
3. **Detect malicious indicators in candidate packages.** Malicious packages often have tells: install scripts that run code (npm `postinstall`, pip `setup.py` executing on install), obfuscated code, network calls on install, or a name-vs-content mismatch. Tools like GuardDog and OSSF package analysis scan for these behaviours:
   ```
   # scan a package for malicious indicators (install scripts, obfuscation, exfil)
   guarddog pypi scan <package>
   ```
4. **Use allowlisting for critical environments.** For high-security builds, allowlist approved packages so an unvetted typosquat can't be installed at all — the strongest control, at the cost of flexibility.
5. **Pin and review dependencies.** Lockfiles pin exact packages/versions (lockfile-integrity); review dependency additions in pull requests so a wrong or suspicious name gets caught by a human before it merges.
6. **Monitor for squats of *your* packages.** If you publish packages, watch for typosquats of *your* names targeting *your* users, and report them for takedown.
7. **Report and remove malicious packages** you find (the malicious-package-response skill) — reporting to the registry protects the whole community.

### Cheatsheet

```
attack: malicious package with name CLOSE to a popular one -> typo/mis-copy pulls it in
  reqeusts~requests | loadsh~lodash | python3-dateutil~python-dateutil

variants: TYPOsquat (keystroke off) | COMBOsquat (plausible extra word) | ecosystem confusion

defend
  VET before adding: exact correct name? genuine popular pkg? (downloads, maintainer, repo, age)
    red flag: brand-new pkg, name close to a popular one, few downloads
  DETECT malicious indicators (GuardDog / OSSF analysis)
    install scripts running code (postinstall / setup.py) | obfuscation | install-time network | name-vs-content mismatch
    guarddog pypi scan <pkg>
  ALLOWLIST approved packages for critical builds (strongest, less flexible)
  PIN (lockfile) + REVIEW dependency additions in PRs (human catches wrong name)
  MONITOR squats of YOUR packages ; REPORT malicious ones (protects community)
```

### Reading the risk

- **A newly-published package with a name one keystroke off a popular one and few downloads** = a classic typosquat red flag; verify the exact correct name against the genuine package before adopting. The core detection.
- **A candidate package with an install script running code** (`postinstall`, `setup.py` executing) = a common malicious-package tell; legitimate packages rarely need to run arbitrary code on install. A strong indicator to investigate.
- **Obfuscated code or install-time network calls** in a package = suspicious behaviour typical of malicious squats; scanners (GuardDog) flag these. Don't install.
- **A dependency added by PR with a subtly-wrong name** = exactly what review catches; a human noticing `reqeusts` vs `requests` stops it before merge. Review dependency additions.
- **A typosquat of your *own* package** = a threat to your users; monitor for lookalikes of your names and report them.
- **Vetted, pinned, reviewed dependencies from allowlisted sources** = the intake protected against name-confusion attacks.

### Pitfalls

- **Not verifying the exact package name.** A single typo or mis-copied name pulls in the squat; verify against the genuine popular package (downloads, maintainer, repo) before adopting. The whole attack relies on this slip.
- **Ignoring install scripts.** Malicious packages run code on install (postinstall/setup.py); a package that executes arbitrary code on install is a red flag. Scan for it.
- **No review of dependency additions.** Without a human checking new dependencies, a wrong or suspicious name merges silently; review them in PRs.
- **Trusting a name because it's plausible.** Combosquats (`python-requests`) look reasonable but may be malicious; verify it's the genuine package, not just a plausible name.
- **Not monitoring your own package names.** Typosquats of your packages target your users; watch for and report them.

### References

- GuardDog, OSSF package analysis, and Socket.dev (malicious-package detection)
- The dependency-confusion, lockfile-integrity, and malicious-package-response skills
- npm/PyPI security and reporting documentation
- OpenSSF supply-chain security guidance

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.