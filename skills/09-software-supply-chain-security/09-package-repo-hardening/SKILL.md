---
format: "v2"
name: "package-repo-hardening"
title: "Package Repo Hardening"
title_fr: "Durcissement des dépôts de paquets"
description: "Use when securing internal package registries — the repositories that serve your organisation's dependencies, so they can't be abused to distribute malicious or confused packages."
description_fr: "À utiliser pour sécuriser les registres de paquets internes — les dépôts qui fournissent les dépendances de l'organisation — afin qu'ils ne puissent pas être détournés pour distribuer des paquets malveillants ou victimes de confusion de noms."
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

Internal package registries (Artifactory, Nexus, a private npm/PyPI) are the distribution point for your organisation's dependencies — and a chokepoint that, if hardened, controls what code enters your builds, or, if weak, becomes a way to distribute malicious packages to every project that pulls from it. This skill covers hardening the package registry so it's a control point for supply-chain security rather than a weak link. It's where several other supply-chain defences are enforced.

### When to use it

Running an internal registry (common in any organisation of size), and as a central place to enforce supply-chain controls. A hardened registry is force-multiplying: it applies defences to every project that consumes from it, rather than relying on each project to do it individually.

### Procedure

1. **Control what enters the registry.** The registry should only serve vetted packages — scan incoming packages for vulnerabilities and malware (the scanning and typosquat skills), and don't blindly proxy everything from public registries. An unfiltered proxy passes malicious public packages straight to your builds.
2. **Prevent dependency confusion at the registry — a key role.** Configure the registry so internal package names always resolve to internal packages, never shadowed by a higher-versioned public one (the dependency-confusion skill). A virtual/proxy registry with correct resolution priority is the central place to close confusion for all projects.
3. **Restrict who can publish.** Publishing to the registry is high-trust — a malicious or compromised publisher distributes to everyone who pulls. Require authentication, scope publish permissions tightly (least privilege), and control who can push to which namespaces. An open-publish registry is a distribution vector for an attacker.
4. **Authenticate and authorise consumers.** Require authentication to pull, and use RBAC so projects/teams access only the packages they should. This limits blast radius and provides an audit trail.
5. **Enforce integrity.** Serve packages with integrity hashes, support signing/verification (the signing skill), and ensure lockfile integrity works against the registry so consumers get exactly what they expect.
6. **Log and monitor registry activity.** Publishing, unusual pulls, and permission changes are security-relevant; log them and alert on anomalies (an unexpected publish, a package being overwritten). The registry is a high-value target.
7. **Harden and patch the registry itself.** It's an internet-adjacent service holding your software supply chain; harden it as infrastructure (the host/network domains), keep it patched, and protect its admin access.

### Cheatsheet

```
internal registry = distribution point for your deps + a CHOKEPOINT
  hardened -> control what enters your builds ; weak -> distributes malicious to EVERY project
  force-multiplying: enforce defences ONCE for all consumers

harden
  CONTROL INTAKE: only vetted packages ; scan incoming (vuln + malware) ; don't blindly proxy public
  DEPENDENCY CONFUSION (key role): internal names always resolve internal, never shadowed by public
    -> the central place to close confusion for all projects
  RESTRICT PUBLISH (high-trust): auth required, least-privilege, namespace control
    (open publish = distribution vector for an attacker)
  CONSUMERS: auth to pull + RBAC (least access + audit trail)
  INTEGRITY: hashes + signing/verification + lockfile integrity works against it
  LOG + monitor (publish, unusual pulls, overwrite, permission changes) — high-value target
  HARDEN + patch the registry itself (infra + admin access)
```

### Reading the setup

- **A registry that blindly proxies everything from public registries** = malicious public packages pass straight to your builds; the registry should vet (scan) intake, not just relay. An unfiltered proxy is a weak link, not a control.
- **Internal names resolvable from public through the registry** = dependency confusion open for every project; the registry is the central place to enforce internal-names-resolve-internal. Getting this right closes confusion org-wide at once.
- **Open or loosely-controlled publish access** = a malicious or compromised publisher distributes to everyone pulling from the registry; publish is the highest-trust operation. Restrict it tightly.
- **Unauthenticated pulls / no consumer RBAC** = no access control or audit trail; anyone can pull anything, and blast radius is unbounded. Authenticate and scope.
- **No integrity/signing support** = consumers can't verify they got the genuine package; the registry should serve hashes and support signature verification.
- **A vetted-intake, confusion-safe, publish-restricted, authenticated, integrity-enforcing, monitored registry** = a supply-chain control point that protects every consuming project.

### The fix / best practice

- **Vet what enters** — scan incoming packages for vulnerabilities and malware; don't blindly proxy public registries.
- **Close dependency confusion at the registry** — internal names always resolve internal; this protects all projects centrally.
- **Restrict publish** to authenticated, least-privilege publishers with namespace control.
- **Authenticate and RBAC consumers** for access control and audit.
- **Enforce integrity** — hashes, signing/verification, and working lockfile integrity.
- **Log, monitor, harden, and patch** the registry as the high-value supply-chain target it is.

### Pitfalls

- **Blindly proxying public registries.** An unfiltered proxy passes malicious public packages straight to your builds; vet (scan) intake. The registry should be a control, not a relay.
- **Not closing dependency confusion at the registry.** It's the central place to ensure internal names resolve internal for all projects; leaving it to each project is error-prone.
- **Open publish access.** Publishing distributes to everyone; a weak-controlled publish is an attacker's distribution vector. Restrict tightly.
- **No consumer authentication/RBAC.** Unbounded access and no audit trail; authenticate pulls and scope access.
- **Neglecting the registry as infrastructure.** It holds your software supply chain and is a high-value target; harden, patch, monitor, and protect admin access.

### References

- Artifactory and Nexus security/hardening documentation
- The dependency-confusion, typosquat-detection, artifact-signing-sigstore, and lockfile-integrity skills
- The devsecops dependency-scanning skill (scanning intake)
- OpenSSF and NIST SP 800-161 supply-chain guidance

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.