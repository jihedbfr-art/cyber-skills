---
format: "v2"
name: "artifact-integrity"
title: "Artifact Integrity"
title_fr: "Intégrité des artefacts"
description: "Use when ensuring build artifacts aren't tampered with between build and deploy — signing outputs and verifying them before deployment so only trusted builds run."
description_fr: "À utiliser pour garantir que les artefacts de build ne sont pas altérés entre la construction et le déploiement — en signant les sorties et en les vérifiant avant déploiement pour que seuls des builds de confiance s'exécutent."
domain: "08-devsecops-and-cicd-security"
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

A build produces an artifact — a binary, a container image, a package — that travels from the build system through a registry to production. Anywhere along that path, the artifact could be swapped or tampered with, and without verification you'd deploy the malicious version trusting it's yours. Artifact integrity ensures only genuine, unmodified build outputs get deployed, through signing and verification. This skill covers protecting the artifact between build and deploy, the general case of the container image-signing skill.

### When to use it

Securing the build-to-deploy path for any artifact type (binaries, packages, images). It's a core supply-chain control — the SolarWinds attack was fundamentally an artifact-integrity failure (malicious code injected into signed builds). It pairs with pipeline hardening and build provenance.

### Procedure

1. **Understand the threat: tampering between build and deploy.** After the build, the artifact sits in a registry/repository and travels to production. An attacker who compromises the registry, the transport, or the build itself can substitute a malicious artifact. Without integrity verification, you deploy whatever's there, trusting the name. Signing and verification break this.
2. **Sign artifacts at build time.** Cryptographically sign each build output (Sigstore/cosign for images and blobs, or language/package signing) so its integrity and origin can be verified. Sign the immutable artifact (by digest/hash), tying the signature to your build identity.
3. **Verify signatures before deploy — the enforcement.** The deployment step (or admission control for images) must verify the artifact's signature against a trusted key before running it, rejecting unsigned or tampered artifacts. Signing without verification protects nothing; the verification gate is what makes it real.
4. **Protect the signing keys.** The signing key is the crown jewel — if an attacker gets it, they sign malicious artifacts that pass verification. Use keyless signing (Sigstore, tied to workload identity) or well-protected keys (HSM/KMS), and never expose signing keys to untrusted pipeline stages.
5. **Verify integrity by hash/digest throughout.** Reference artifacts by immutable digest, not mutable tags/versions, so you deploy exactly the artifact you built and signed (the same digest-pinning as the image supply-chain skill). A mutable reference can be repointed to a tampered artifact.
6. **Attach attestations.** Beyond a signature, attach attestations about the artifact (in-toto, SLSA provenance — the build-provenance skill): how it was built, what went in, that it passed the pipeline's checks. Verification can then check not just "is it signed" but "was it built the way it should be".
7. **Automate the full chain.** Build → sign → (registry) → verify → deploy, all automated and enforced so no unsigned or unverified artifact can reach production. Manual signing/verification gets skipped.

### Cheatsheet

```
artifact travels build -> registry -> production ; can be SWAPPED/tampered anywhere
  without verification you deploy whatever's there, trusting the name (SolarWinds = this failure)

1. SIGN at build (Sigstore/cosign, or package signing) — sign the immutable artifact (digest/hash)
2. VERIFY before deploy (the enforcement): deploy step / admission verifies signature vs trusted key
     -> reject unsigned/tampered. signing WITHOUT verification protects nothing.
3. PROTECT signing keys (crown jewel — stolen key = signed malicious artifacts)
     keyless (Sigstore, workload identity) or HSM/KMS ; never expose to untrusted stages
4. reference by DIGEST not mutable tag (repointable to tampered artifact)
5. ATTESTATIONS (in-toto / SLSA provenance): how built + what went in + passed checks
     -> verify "built correctly", not just "signed"
6. AUTOMATE build->sign->verify->deploy (manual = skipped)
```

### Reading the setup

- **Artifacts deployed with no integrity verification** = you're trusting that whatever's in the registry is your genuine build; a compromised registry or tampered artifact deploys unchecked. Signing plus verification closes this — the control that would have mattered in SolarWinds-class attacks.
- **Signing without a verification gate** = signatures nobody checks; the value is entirely in the deploy-time verification that rejects unsigned/tampered artifacts. They must go together.
- **Signing keys exposed to untrusted pipeline stages** = an attacker who gets the key signs malicious artifacts that pass verification, defeating the whole scheme. Protect keys (keyless/HSM) and isolate them.
- **Mutable artifact references** (deploy by tag/version) = the reference can be repointed to a tampered artifact after signing; deploy by immutable digest.
- **Signature-only verification** (no attestation) = confirms origin but not that the build wasn't subverted; attestations verify it was built correctly. The stronger check.
- **Automated build→sign→verify→deploy with protected keys and attestations** = artifact integrity assured; only genuine, correctly-built artifacts run.

### The fix / best practice

- **Sign artifacts at build** (Sigstore/cosign or package signing), signing the immutable digest.
- **Enforce signature verification before deploy** — reject unsigned/tampered artifacts at the deploy gate or admission controller.
- **Protect signing keys** with keyless signing or HSM/KMS, isolated from untrusted pipeline stages.
- **Reference by digest, not mutable tags/versions**, to deploy exactly what you signed.
- **Attach and verify attestations** (SLSA provenance, in-toto) so verification checks the build's integrity, not just the signature.
- **Automate the whole chain** so no unsigned/unverified artifact can deploy.

### Pitfalls

- **No verification.** Signing without a deploy-time verification gate protects nothing; the enforcement is what rejects tampered artifacts. Verify before deploy.
- **Exposed signing keys.** A stolen key lets attackers sign malicious artifacts that pass verification — the whole scheme collapses. Use keyless/HSM and isolate keys.
- **Mutable references.** Deploying by tag/version lets the reference be repointed to a tampered artifact after signing; pin by digest.
- **Signature without attestation.** It confirms origin but not that the build wasn't subverted (SolarWinds signed its malicious builds); add provenance attestations.
- **Manual signing/verification.** It gets skipped; automate and enforce the chain.

### References

- Sigstore/cosign, in-toto, and SLSA framework documentation
- The container supply-chain-for-images skill (the container-specific case) and pipeline-hardening skill
- The software-supply-chain-security domain (artifact-signing, build-provenance)
- The SolarWinds attack (canonical artifact-integrity failure)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.