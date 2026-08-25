---
format: "v2"
name: "dockerfile-hardening"
title: "Dockerfile Hardening"
title_fr: "Durcissement du Dockerfile"
description: "Use when writing or reviewing a Dockerfile for security — non-root users, minimal base images, no secrets in layers, and the build practices that shrink the attack surface."
description_fr: "À utiliser pour écrire ou relire un Dockerfile sous l'angle sécurité — utilisateur non-root, image de base minimale, absence de secrets dans les layers, et les pratiques de build qui réduisent la surface d'attaque."
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

The Dockerfile decides what's in your container and how it runs — and a few habits separate a hardened image from a bloated, root-running, secret-leaking one. This skill covers the Dockerfile practices that shrink the attack surface: minimal bases, non-root execution, no secrets baked into layers, and pinned, verifiable dependencies. It's where container security starts, before the image ever runs.

### When to use it

Writing or reviewing any Dockerfile, and as a baseline for the whole container programme. It pairs with image scanning (which finds vulnerabilities) — hardening reduces what's there to be vulnerable in the first place.

### Procedure

1. **Start from a minimal base image — the highest-impact choice.** Every package in the base is attack surface and a potential CVE. Prefer distroless, alpine, or slim variants over full OS images. A smaller base means fewer vulnerabilities, a smaller image, and less for an attacker to use (fewer shells and tools inside).
2. **Run as a non-root user.** By default containers run as root, so a container escape or app compromise is root. Create and switch to a non-root user (`USER`), so the process has minimal privilege even if compromised. This is one of the most important hardening steps.
3. **Keep secrets out of the image — they persist in layers.** Never `COPY` a credential or `ENV` a secret; Docker layers are inspectable, so a secret added in one layer remains even if "removed" in a later one. Use build secrets (BuildKit `--secret`) or inject at runtime, never bake them in.
4. **Pin and verify what you install.** Pin base image tags (ideally by digest, not a moving tag) and package versions so builds are reproducible and you know what you shipped. Verify downloaded artifacts where possible.
5. **Minimise layers and content.** Combine related `RUN` commands, remove build tools and caches in the same layer they're used (a package cache left in an earlier layer persists), and use `.dockerignore` so you don't copy secrets, `.git`, or junk into the image. Multi-stage builds keep build-time tools out of the final image.
6. **Drop unnecessary capabilities and set a read-only filesystem** where possible (often at runtime, but design for it) — the container should have only what it needs.
7. **Lint and scan the result.** Use a Dockerfile linter (hadolint) to catch bad practices and a scanner (Trivy — the image-scanning skill) on the built image. Automate both in CI.

### Cheatsheet

```
the Dockerfile decides what's IN the container + how it RUNS. harden it.

1. MINIMAL BASE (highest impact): distroless / alpine / slim > full OS
     fewer packages = fewer CVEs + smaller + fewer tools for an attacker
2. NON-ROOT: create + USER a non-root user (default is root -> escape = root)
3. NO SECRETS IN LAYERS: never COPY creds / ENV secrets
     layers are inspectable — "removed" in a later layer STILL persists
     -> BuildKit --secret / runtime injection
4. PIN + VERIFY: base by digest (not moving tag), package versions -> reproducible
5. MINIMISE: combine RUNs, remove build tools/caches in the SAME layer,
     .dockerignore (no .git/secrets/junk), MULTI-STAGE build (build tools out of final)
6. drop capabilities + read-only filesystem (design for it)
7. LINT (hadolint) + SCAN (trivy) in CI

quick check: does it run as root? secrets in layers? full-fat base? unpinned?
```

### Reading a Dockerfile

- **A full-OS base image** = large attack surface and many inherited CVEs; switching to distroless/alpine/slim removes most of them at once. The single highest-impact hardening choice.
- **No `USER` directive (runs as root)** = a container escape or app compromise is immediately root; a non-root user is one of the most important mitigations. A common, high-value finding.
- **A secret `COPY`'d or `ENV`'d** = it persists in the image layers, extractable by anyone who pulls the image, even if a later layer "deletes" it. A direct credential leak — the layer history is inspectable.
- **Unpinned base tags / package versions** = non-reproducible builds; you can't be sure what you shipped, and `latest` can change under you. Pin by digest.
- **Build tools and caches in the final image** = unnecessary bloat and attack surface (compilers, package managers an attacker can use); multi-stage builds and same-layer cleanup remove them.
- **A minimal, non-root, secret-free, pinned, multi-stage image** = the hardened baseline; scanning it should find far less.

### The fix / best practice

- **Minimal base + non-root user + no baked secrets** are the three highest-value habits; adopt them as defaults.
- **Multi-stage builds** to keep build-time tooling out of the runtime image.
- **Pin base images by digest and pin package versions** for reproducibility.
- **BuildKit secrets or runtime injection** for anything sensitive; never in layers.
- **`.dockerignore`** to avoid copying `.git`, secrets, and junk.
- **Lint (hadolint) and scan (Trivy) in CI**, failing the build on bad practices and fixable high/critical vulnerabilities.

### Pitfalls

- **Running as root.** The default, and a major mitigation missed; escape or compromise becomes root. Add a non-root `USER`.
- **Secrets in layers.** `COPY`/`ENV` of a credential persists in the inspectable layer history even if "removed" later — a real leak. Use build secrets or runtime injection.
- **Fat base images.** A full OS base inherits huge CVE surface and gives attackers tools/shells; use minimal bases.
- **Unpinned tags.** `latest` and unpinned packages make builds non-reproducible and can change silently; pin by digest.
- **Leaving build tooling in the final image.** Compilers and package managers are attack surface; use multi-stage builds.
- **Not linting/scanning.** Bad practices and vulnerabilities ship silently; automate hadolint and Trivy in CI.

### References

- Docker security best practices and Dockerfile reference
- hadolint (Dockerfile linter) and Trivy (image scanner) documentation
- CIS Docker Benchmark
- The container-image-scanning and supply-chain-for-images skills

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.