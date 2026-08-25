---
format: "v2"
name: "dependency-and-config-review"
title: "Dependency And Config Review"
title_fr: "Revue des dépendances et de la configuration"
description: "Use when reviewing dependencies and configuration for the insecure default nobody changed — vulnerable libraries, dangerous framework settings, and debug flags left on."
description_fr: "À utiliser lors de la revue des dépendances et de la configuration, à la recherche du réglage par défaut non sécurisé que personne n'a modifié — bibliothèques vulnérables, paramètres de framework dangereux, indicateurs de débogage laissés actifs."
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

A lot of real-world compromise doesn't come from the code someone wrote — it comes from a library with a known CVE, or a framework setting left at a convenient-but-unsafe default. This pass reviews the parts of the repo that aren't application logic: the manifests, the config, the framework wiring. It's less about cleverness and more about knowing which defaults bite.

### When to use it

On dependency bumps, new-dependency additions, and config changes — and as a standing sweep, since CVEs land on dependencies you already have without any diff on your side. Pair it with the software-supply-chain domain for the deeper provenance angle; this skill is the reviewer's fast pass.

### Two fronts

**Dependencies.** The questions are: is anything here on a version with a known vuln, is it actually reachable, and is the version pinned? A scanner answers the first; you answer the second.

```bash
osv-scanner scan .
npm audit --omit=dev        # node
pip-audit                   # python
mvn org.owasp:dependency-check:check   # java
```

Don't stop at the scanner's number. A critical CVE in a transitive dep you never call the vulnerable function of is lower risk than a medium in your hot path — note reachability, not just severity. And check for unpinned ranges (`^`, `~`, `latest`, no lockfile): those turn a future malicious release into your problem automatically.

**Configuration and framework defaults.** This is where the quiet ones hide. The pattern to internalise: *frameworks default to developer-friendly, not production-safe.* Some recurring ones:

- **Debug mode on in production.** `DEBUG=True` (Django), `app.debug`, detailed error pages, dev profiles active. Leaks internals and sometimes enables code execution.
- **Default or example credentials** in config — admin/admin, sample keys, the framework's demo secret. Overlaps with secrets-in-code; here it's the *default* nobody changed.
- **Permissive CORS.** `Access-Control-Allow-Origin: *` combined with credentials, or reflecting the Origin header unchecked.
- **Disabled security features** — CSRF protection off, TLS verification disabled (`verify=False`, `rejectUnauthorized:false`), auto-escaping turned off in a template engine, actuator/management endpoints exposed unauthenticated.
- **Overbroad defaults** — a wildcard host allowlist, a bucket/dir served with directory listing, a message broker or database bound to `0.0.0.0` with no auth.

The Spring Boot actuator one catches teams often: `/actuator` handy in dev, and someone forgets it's reachable in prod exposing `/env`, `/heapdump`, sometimes `/jolokia` straight to RCE. Grep the management config and the security config together — one without the other tells you half the story.

### Procedure

1. Run a dependency scanner; for each hit, judge severity *and* reachability, and check version pinning / lockfile presence.
2. Read the production config and profiles. For each security-relevant setting, ask "is this the safe value or the convenient default?"
3. Grep the specific footguns (below) and confirm the prod value, not just that the setting exists.
4. Separate "must fix" (debug on, TLS verify off, exposed admin endpoint, exploitable CVE in reach) from "should improve" (unpinned versions, low-severity transitive CVE) so the report is actionable.

### Cheatsheet

```bash
rg -ni 'debug\s*[:=]\s*true|app\.debug|display_errors\s*=\s*On|NODE_ENV.*development'
rg -ni 'verify\s*=\s*false|rejectUnauthorized:\s*false|csrf.*disable|autoescape\s*=\s*false'
rg -ni 'Allow-Origin.{0,4}\*|0\.0\.0\.0|management\.endpoints|actuator|allowed_hosts.*\*'
rg -n '"\^|"~|:latest|version: *"\*"'
```

### Reading it

- **Exploitable, reachable CVE** → fix now (upgrade/patch). Reachable beats severe-but-unreachable.
- **Debug/dev profile or `verify=false` in a prod config** → finding regardless of anything else; it undoes other controls.
- **Exposed management/admin endpoint without auth** → treat as critical; it's often a direct path in.
- **Unpinned versions / missing lockfile** → not an active vuln but a supply-chain exposure; recommend pinning.
- **A CVE in a dev-only dependency** → real but lower; note the scope so it's triaged right.

### The fix

Upgrade or patch reachable-vulnerable dependencies; pin versions and commit a lockfile so releases are deterministic. Set production config explicitly to the safe values rather than inheriting framework defaults — debug off, TLS verification on, CSRF on, auto-escaping on, management endpoints authenticated and scoped, CORS to a real allowlist. Automate the dependency side (Dependabot/Renovate + a scanner in CI) so this stops being a manual memory exercise.

### Pitfalls

- **Severity tunnel vision.** A reachable medium can outrank an unreachable critical. Factor in whether you call the vulnerable path.
- **Reviewing the setting's existence, not its prod value.** Config often differs per profile; check the one that ships.
- **Trusting defaults because they "came with the framework."** The default is chosen for onboarding ease, not your threat model.
- **Ignoring pinning.** An unpinned dep is a standing invitation for a future supply-chain hit.

### References

- OWASP A05:2021 Security Misconfiguration, A06:2021 Vulnerable and Outdated Components
- CWE-1104 (unmaintained third-party components), CWE-16 (configuration), CWE-1188 (insecure default)
- osv-scanner, Dependabot/Renovate; see also domain 09 software-supply-chain-security

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.