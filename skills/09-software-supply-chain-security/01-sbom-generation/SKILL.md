---
name: sbom-generation
domain: 09-software-supply-chain-security
description: Use when you need a complete inventory of what's inside a build — every dependency and its version — as an SBOM you can scan, track, and hand to auditors.
difficulty: beginner
tags: [supply-chain, sbom, inventory, dependencies, compliance]
tools: [syft, trivy, cyclonedx]
---

## Purpose

You can't defend a supply chain you can't enumerate. A Software Bill of Materials is the ingredient list — every component, version, and licence in an artifact — in a standard format. When the next Log4Shell drops, an SBOM answers "are we affected, and where" in seconds instead of a frantic week. This skill covers generating one and putting it to use.

## When to use it

Generate an SBOM as a build output for anything you ship or run: application artifacts, container images, releases. It underpins vulnerability triage, licence compliance, and incident response, so it's the foundation for most of this domain.

## Procedure

1. Generate the SBOM from the artifact. Syft reads a container image, directory, or archive and lists everything it finds:
   ```
   syft myapp:latest -o cyclonedx-json > sbom.json
   ```
   Prefer a standard format — **CycloneDX** or **SPDX** — so other tools can consume it.
2. Generate it from the **built artifact**, not just the source manifest. A lockfile lists declared deps; scanning the built image/binary captures what actually shipped, including transitive and system packages.
3. Store the SBOM as a versioned build artifact, tied to the exact release it describes. An SBOM that doesn't match a specific build is worthless during an incident.
4. Feed it to a scanner to turn the inventory into a risk view — the SBOM says what's there, the scanner says what's vulnerable:
   ```
   trivy sbom sbom.json
   ```
5. Diff SBOMs across releases to see what changed — a new dependency appearing is a supply-chain event worth noticing.
6. When a new critical CVE lands, query your stored SBOMs for the affected component instead of re-scanning everything from scratch.

## Cheatsheet

```bash
# generate (image, dir, or archive)
syft myapp:latest -o cyclonedx-json > sbom.json
syft dir:. -o spdx-json > sbom.spdx.json

# generate with trivy instead
trivy image --format cyclonedx -o sbom.json myapp:latest

# use it: scan the SBOM for vulns
trivy sbom sbom.json
grype sbom:sbom.json

# answer "am I affected by CVE-X?"
grep -i 'log4j' sbom.json
```

## Reading the output

- **The component count and depth** — a huge transitive tree is a large attack surface; it's worth knowing before an incident, not during one.
- **Unexpected components** you didn't knowingly pull in are the supply-chain signal — a package appearing that no one added deserves a look.
- **Version accuracy** matters most: the SBOM's value is answering "which version of X shipped", so verify it reflects the built artifact, not a stale manifest.
- **Licences** in the SBOM surface compliance risk (a copyleft dependency in a proprietary product) alongside the security view.

## How to make it useful (the "fix")

An SBOM is only worth generating if it's used:

- **Automate it in CI** as a build artifact for every release, in a standard format (CycloneDX/SPDX).
- **Store and index** SBOMs so you can query across your whole estate when a CVE drops — that fast lookup is the entire payoff.
- **Scan continuously**, not once — the inventory is static, but new vulnerabilities against those components appear over time.
- **Diff releases** to catch new or changed dependencies early.
- Combine with signing/provenance (later skills in this domain) so you can also trust the SBOM reflects a build nobody tampered with.

## Pitfalls

- **Generating from source manifests only.** Misses transitive and OS-level components that the built artifact actually contains.
- **A one-off SBOM.** Generated once and never regenerated, it drifts from reality and misleads during an incident.
- **SBOM without scanning.** An inventory nobody checks against vulnerability data is a document, not a control.
- **No version linkage.** An SBOM not tied to a specific build can't answer the one question you'll ask it under pressure.

## References

- CycloneDX and SPDX specifications
- Syft, Trivy, and Grype documentation
- CISA — Software Bill of Materials guidance
- NTIA — Minimum Elements for an SBOM
