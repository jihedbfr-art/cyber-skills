---
name: build-provenance-slsa
domain: 08-devsecops-and-cicd-security
description: Use when establishing verifiable build provenance — proving how and where an artifact was built using the SLSA framework, so consumers can trust the build wasn't subverted.
difficulty: advanced
tags: [devsecops, slsa, provenance, supply-chain, attestation]
tools: [slsa, sigstore, in-toto]
---

## Purpose

Signing an artifact proves *who* published it; build provenance proves *how it was built* — that it came from the expected source, through the expected pipeline, without tampering. This matters because supply-chain attacks (SolarWinds) subverted the *build process* itself, producing malicious artifacts that were then legitimately signed. SLSA (Supply-chain Levels for Software Artifacts) is the framework for establishing and verifying that provenance. This skill covers build provenance and SLSA, the top of the supply-chain-integrity stack.

## When to use it

Maturing supply-chain security beyond signing to provenance, for organisations that need strong assurance their builds weren't subverted (or that produce software others depend on). It builds on artifact integrity and pipeline hardening — provenance is what you attest and verify once those are in place.

## Procedure

1. **Understand what provenance proves — beyond signing.** A signature says "I published this"; provenance says "this artifact was built from *this* source, by *this* builder, through *this* process, with *these* inputs". It catches build subversion: a signed artifact from a compromised build is still malicious, but its provenance would reveal it wasn't built as expected. Provenance is the answer to the SolarWinds class of attack.
2. **Understand the SLSA levels.** SLSA defines increasing levels of build-integrity assurance:
   - **L1** — provenance exists (the build documents how it was made).
   - **L2** — provenance is signed and the build runs on a hosted build service.
   - **L3** — the build is hardened and isolated (tamper-resistant build, non-falsifiable provenance).
   Higher levels mean stronger guarantees that the build wasn't tampered with. Aim for the level your risk warrants.
3. **Generate provenance in the build.** Have the build system produce a signed provenance attestation (in-toto format, via SLSA generators for common CI systems) recording the source, builder identity, build parameters, and inputs. Modern CI platforms and the SLSA project provide generators that do this.
4. **Use a trustworthy, isolated build.** Provenance is only as trustworthy as the build that produced it — a compromised build can lie in its provenance. SLSA's higher levels require the build to run on a hardened, isolated service (the pipeline-hardening and secure-runners skills) so the provenance is non-falsifiable.
5. **Verify provenance before deploy/consume.** The consumer (your deployment, or a downstream user) verifies the provenance: was it built from the expected source, by the expected builder, meeting the required SLSA level? Reject artifacts whose provenance doesn't match. Verification is what makes provenance useful — generated-but-unverified provenance protects nothing.
6. **Combine with signing and SBOM.** Provenance (how built), signing (who published), and SBOM (what's inside) together give full supply-chain assurance — origin, integrity, build process, and contents all verifiable.
7. **Automate generation and verification** in the pipeline so every artifact carries provenance and every deployment checks it.

## Cheatsheet

```
signing = WHO published ; provenance = HOW it was built (source, builder, process, inputs)
  catches BUILD SUBVERSION (SolarWinds: compromised build -> malicious artifact, legitimately signed)
  SLSA = framework for establishing + verifying provenance

SLSA levels (increasing build-integrity assurance)
  L1  provenance exists (documents how built)
  L2  provenance SIGNED + hosted build service
  L3  hardened + isolated build (tamper-resistant, NON-FALSIFIABLE provenance)
  -> aim for the level your risk warrants

1. GENERATE provenance in build (in-toto attestation, SLSA generators for CI)
     records source + builder identity + build params + inputs
2. TRUSTWORTHY ISOLATED build (compromised build can LIE in provenance)
     higher SLSA = hardened isolated builder (pipeline-hardening, secure-runners)
3. VERIFY before deploy/consume: expected source? expected builder? required level?
     -> reject mismatches. unverified provenance protects nothing.
combine: provenance (how) + signing (who) + SBOM (what) = full assurance
automate generation + verification
```

## Reading the provenance

- **Signing without provenance** = you know who published, but not that the build wasn't subverted; a compromised build produces malicious artifacts that sign legitimately (SolarWinds). Provenance is what catches build-process subversion — the gap signing alone leaves.
- **Provenance generated but never verified** = protects nothing; the value is in the deploy/consume-time verification that the artifact was built from the expected source by the expected builder. Generation without verification is theatre.
- **Provenance from a non-isolated build** = a compromised build can lie in its provenance; low SLSA levels give weaker guarantees. Higher levels require a hardened, isolated builder so the provenance is non-falsifiable.
- **The target SLSA level vs your risk** = higher levels cost more (isolated builds, verification infrastructure) but give stronger guarantees; aim for the level your threat model warrants, not blindly L3.
- **Provenance + signing + SBOM together** = full supply-chain assurance (how built, who published, what's inside, all verifiable) — the complete picture.
- **Automated provenance generation and verification** = every artifact provably built as expected; the top of the supply-chain-integrity stack.

## Pitfalls

- **Relying on signing alone.** Signing proves publisher, not build integrity; a subverted build produces legitimately-signed malicious artifacts (SolarWinds). Add provenance to catch build subversion.
- **Generating provenance without verifying it.** Unverified provenance protects nothing; the deploy/consume-time check is what makes it useful. Verify against expected source/builder/level.
- **Provenance from an untrustworthy build.** A compromised build lies in its provenance; higher SLSA levels require isolated, hardened builds so provenance is non-falsifiable.
- **Chasing the highest level blindly.** SLSA L3 costs real effort; match the level to your risk rather than over-investing or under-protecting.
- **Provenance without signing and SBOM.** Full assurance needs all three (how/who/what); provenance alone is incomplete.

## References

- SLSA framework (slsa.dev) and in-toto attestation specifications
- Sigstore, SLSA provenance generators for common CI systems
- The artifact-integrity, pipeline-hardening, and secure-runners skills
- The software-supply-chain-security domain (sbom-generation, build-provenance) and the SolarWinds attack
