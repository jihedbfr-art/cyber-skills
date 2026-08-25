---
format: "v2"
name: "build-provenance"
title: "Build Provenance"
title_fr: "Provenance de build"
description: "Use when attesting where and how software was built — generating build provenance so consumers can verify an artifact came from the expected source and pipeline, uncompromised."
description_fr: "À utiliser pour attester où et comment un logiciel a été construit — générer une provenance de build permettant aux consommateurs de vérifier qu'un artefact provient bien de la source et du pipeline attendus, sans compromission."
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

Build provenance is verifiable metadata about *how* an artifact was produced — the source it was built from, the builder that built it, the process and inputs. It's what lets a consumer trust that an artifact wasn't subverted during the build, catching the class of attack (SolarWinds) where the build process itself is compromised to produce malicious-but-legitimately-signed artifacts. This skill covers generating and using build provenance in the supply-chain context, complementing the DevSecOps SLSA skill from the supply-chain-integrity angle.

### When to use it

Producing software others depend on (or consuming software where build integrity matters). It's the "how was this built" layer of supply-chain assurance, sitting alongside signing (who published) and SBOM (what's inside). It builds on artifact signing and connects to the DevSecOps build-provenance-slsa skill.

### Procedure

1. **Understand what provenance adds beyond signing and SBOM.** Signing proves the publisher; an SBOM lists the contents; provenance proves the *build process* — that the artifact came from the expected source, through the expected builder, with the expected inputs. Together they answer who/what/how. Provenance is the piece that catches build subversion.
2. **Generate provenance as a signed attestation.** The build produces an attestation (in-toto format, signed via Sigstore) recording: the source repository and commit, the builder identity and platform, the build parameters, and the materials/inputs. SLSA provenance generators for common CI systems produce this automatically.
3. **Tie provenance to a trustworthy build.** Provenance is only as good as the build that generated it — a compromised build can produce false provenance. The stronger guarantees (SLSA higher levels) require the build to run on a hardened, isolated builder so the provenance can't be forged (the pipeline-hardening and secure-runners skills).
4. **Publish provenance with the artifact.** Attach the provenance attestation to the artifact (in the registry, alongside the release) so consumers can retrieve and verify it. Provenance nobody can access is useless.
5. **Verify provenance on consumption — the point.** When you consume an artifact (deploy it, or depend on it), verify its provenance: was it built from the expected source repository, by the expected builder, meeting your required assurance level? Reject artifacts whose provenance doesn't match expectations. Generated-but-unverified provenance protects nothing.
6. **Use provenance for the whole dependency chain, ideally.** The strongest supply-chain security verifies provenance not just for your own builds but for the dependencies you consume — though ecosystem support for this is still maturing. Consuming provenance-attested dependencies raises assurance.
7. **Combine with signing, SBOM, and the other controls** for full supply-chain integrity — provenance (how), signing (who), SBOM (what), integrity hashes (unmodified).

### Cheatsheet

```
provenance = verifiable metadata on HOW an artifact was built (source, builder, process, inputs)
  catches BUILD SUBVERSION (SolarWinds: compromised build -> malicious, legitimately-signed artifact)

the three layers together: SIGN (who published) + SBOM (what's inside) + PROVENANCE (how built)

1. GENERATE as signed attestation (in-toto, via Sigstore ; SLSA generators for CI)
     records: source repo+commit | builder identity+platform | build params | inputs/materials
2. TRUSTWORTHY build (compromised build -> false provenance)
     stronger guarantees (SLSA higher levels) = hardened isolated builder (pipeline-hardening/secure-runners)
3. PUBLISH provenance WITH the artifact (registry/release) — inaccessible = useless
4. VERIFY on consumption (THE POINT): expected source? expected builder? required level?
     -> reject mismatches. unverified provenance protects nothing.
5. ideally verify provenance for CONSUMED dependencies too (ecosystem support maturing)
combine with signing + SBOM + integrity hashes = full assurance
```

### Reading the practice

- **Signing and SBOM without provenance** = you know who published and what's inside, but not that the build wasn't subverted; a compromised build produces malicious artifacts that sign legitimately and have accurate SBOMs. Provenance is the layer that catches build-process compromise.
- **Provenance generated but not published/verifiable** = useless; consumers can't retrieve or check it. Attach it to the artifact and verify on consumption — verification is the whole point.
- **Provenance from a non-isolated build** = a compromised build can forge it; weak provenance gives weak assurance. Higher SLSA levels require hardened, isolated builders so provenance is non-falsifiable.
- **Verification checking only that provenance exists, not its contents** = weak; verify the *expected* source repository and builder, not just presence. An attacker's build has provenance too — just the wrong source/builder.
- **Consuming dependencies with verified provenance** = higher supply-chain assurance than trusting names alone; the maturing frontier of dependency security.
- **Generated, published, and consumption-verified provenance alongside signing and SBOM** = full supply-chain integrity — who, what, and how, all verifiable.

### Pitfalls

- **Relying on signing/SBOM without provenance.** They prove publisher and contents but not build integrity; a subverted build produces legitimately-signed artifacts with accurate SBOMs (SolarWinds). Add provenance.
- **Generating provenance without verifying it.** Unverified provenance protects nothing; the consumption-time verification (expected source/builder/level) is what makes it valuable.
- **Provenance from an untrustworthy build.** A compromised build forges provenance; higher assurance needs isolated, hardened builders.
- **Not publishing provenance with the artifact.** Consumers can't verify what they can't access; attach it.
- **Verifying existence, not contents.** An attacker's build also produces provenance — with the wrong source/builder. Verify the expected values, not just presence.

### References

- SLSA framework (slsa.dev), in-toto attestations, Sigstore
- The devsecops build-provenance-slsa, artifact-integrity skills and this domain's artifact-signing-sigstore, sbom-generation skills
- The SolarWinds attack (canonical build-subversion case)
- OpenSSF supply-chain security guidance

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.