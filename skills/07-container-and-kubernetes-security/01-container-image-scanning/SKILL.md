---
format: "v2"
name: "container-image-scanning"
title: "Container Image Scanning"
title_fr: "Analyse des images de conteneurs"
description: "Use when you need to check a container image for known vulnerabilities, exposed secrets, and bad practice before it ships — and how to keep the results actionable."
description_fr: "À utiliser pour analyser une image de conteneur à la recherche de vulnérabilités connues, de secrets exposés et de mauvaises pratiques avant sa mise en production — et pour transformer les résultats en actions concrètes."
domain: "07-container-and-kubernetes-security"
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

A container image is your code plus a whole OS and every library underneath it. Most of the CVEs in a scan aren't yours — they're in the base image and dependencies you inherited. This skill covers scanning an image for known vulnerabilities and secrets, and, just as importantly, turning the wall of results into the few things worth fixing.

### When to use it

Before pushing an image, in CI on every build, and periodically against images already running (new CVEs get disclosed against images that haven't changed). Also when auditing a base image you're considering standardising on.

### Procedure

1. Scan the built image for known vulnerabilities. Trivy reads the OS packages and language dependencies and matches them against vulnerability databases:
   ```
   trivy image myapp:latest
   ```
2. Filter to what you can act on. Fixed, high/critical vulnerabilities first — an unfixable low is noise, a fixable critical is a task:
   ```
   trivy image --severity HIGH,CRITICAL --ignore-unfixed myapp:latest
   ```
3. Scan for **secrets and misconfiguration** in the image, not just CVEs — a key baked into a layer is worse than most CVEs:
   ```
   trivy image --scanners secret,misconfig myapp:latest
   ```
4. Cross-check with a second scanner where it matters; databases differ and one may flag what the other misses:
   ```
   grype myapp:latest
   ```
5. Trace a finding to its source. Is the vulnerable package from the base image (fix: change/upgrade base) or from your app's dependencies (fix: bump the dependency)? The layer tells you who owns the fix.
6. Wire it into the pipeline with a failing threshold so regressions can't merge (see the DevSecOps dependency-scanning skill for gating strategy).

### Cheatsheet

```bash
trivy image myapp:latest                               # full scan
trivy image --severity HIGH,CRITICAL --ignore-unfixed myapp:latest
trivy image --scanners secret,misconfig myapp:latest    # secrets + dockerfile issues
trivy image -f json -o out.json myapp:latest            # machine-readable for CI
grype myapp:latest                                      # second opinion

trivy config ./Dockerfile
```

### Reading the output

- **Fixed HIGH/CRITICAL CVEs** are the actionable set — there's a patched version to move to. Start here.
- **`--ignore-unfixed` findings that remain** are the real backlog; unfixed CVEs may still matter but you can't patch your way out, so they're a risk-acceptance/mitigation decision.
- **A secret detected in a layer** is a direct finding regardless of severity counts — rotate it and rebuild without it.
- **The same CVE across dozens of images** usually traces to one shared base image; fix the base, fix them all.
- **Severity counts alone are misleading** — a critical in a component your app never invokes may be lower real risk than a high on your network path. Context beats the number (see cvss-in-context).

### The fix

- **Start from a minimal base** (distroless, alpine, or slim). Fewer packages means fewer CVEs and a smaller attack surface — this single choice removes most findings.
- **Rebuild regularly** so base-image patches flow in; a stale image accumulates CVEs without changing a line of your code.
- **Pin and update dependencies** for the vulnerabilities that are actually yours.
- **Keep secrets out of images** — use runtime secret injection, never `COPY` or `ENV` a credential.
- **Gate in CI**: fail the build on fixable HIGH/CRITICAL, and re-scan running images on a schedule.

### Pitfalls

- **Drowning in unfixable lows.** Without `--ignore-unfixed` and a severity filter, the report is unusable and everyone stops reading it.
- **Scanning once at build, never again.** New CVEs land against unchanged images; a clean scan has a short shelf life.
- **Fixing app deps but ignoring the base.** Most CVEs are inherited — a fat base image is the real problem.
- **Treating the count as the risk.** A steered severity filter plus reachability context matters more than "200 vulnerabilities".

### References

- Trivy and Grype documentation
- CIS Docker Benchmark
- NIST SP 800-190 (Application Container Security Guide)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.