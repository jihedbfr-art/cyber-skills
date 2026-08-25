---
format: "v2"
name: "artifact-signing-sigstore"
title: "Artifact Signing Sigstore"
title_fr: "Signature d'artefacts avec Sigstore"
description: "Use when signing and verifying software artifacts with Sigstore/cosign — establishing that artifacts are genuine and unmodified, with keyless signing that removes key-management pain."
description_fr: "À utiliser pour signer et vérifier des artefacts logiciels avec Sigstore/cosign — établir qu'un artefact est authentique et non modifié, grâce à une signature sans clé qui supprime la contrainte de gestion des clés."
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

Signing proves an artifact's origin and integrity — that it came from who you think and wasn't modified. Historically, signing meant painful key management (generating, distributing, and protecting signing keys), which is why so little software was signed. Sigstore changes this with keyless signing tied to identity, making artifact signing practical at scale. This skill covers signing and verifying artifacts with Sigstore/cosign, the modern foundation for supply-chain integrity. It's the general framework behind the container and artifact-integrity skills.

### When to use it

Establishing signing across your software artifacts (containers, binaries, packages, SBOMs, attestations). It's increasingly the standard, and its keyless model removes the main historical barrier — no signing keys to manage or leak.

### How Sigstore works (and why keyless matters)

- **Keyless signing** — instead of managing a long-lived signing key, you authenticate with an identity (OIDC — a CI workload identity, or a developer identity), and Sigstore issues a short-lived certificate tied to that identity for the signing operation. No key to store, protect, or leak. This is the breakthrough that makes signing practical.
- **Transparency log (Rekor)** — signatures are recorded in a public, tamper-evident transparency log, so signing events are auditable and can't be secretly forged/backdated.
- **Verification** checks the signature and that it was made by the expected identity, against the transparency log.

### Procedure

1. **Sign artifacts with cosign.** Sign container images, blobs, or other artifacts. With keyless signing, cosign uses your OIDC identity (in CI, the workload identity) and records the signature in Rekor:
   ```
   cosign sign <artifact>                    # keyless (OIDC identity) — no key to manage
   ```
   Sign the immutable artifact (by digest/hash).
2. **Verify signatures, checking the identity — the enforcement.** Verification isn't just "is it signed" but "was it signed by the *expected* identity" (the specific CI workflow, the specific team). Configure verification to require the correct signer identity and the transparency-log entry:
   ```
   cosign verify --certificate-identity=<expected> --certificate-oidc-issuer=<issuer> <artifact>
   ```
   Enforce this at deploy (admission control) so only artifacts signed by trusted identities run.
3. **Prefer keyless in CI.** For automated builds, keyless signing tied to the CI workload identity is ideal — no key to inject into the pipeline (and thus no key to steal). This removes the "protect the signing key" problem that plagued traditional signing.
4. **Sign more than the artifact.** Sigstore signs attestations too — SBOMs (what's inside), provenance (how built, SLSA), and vulnerability scan results — so you can verify all of these, not just the artifact's existence.
5. **Use the transparency log for auditability.** Rekor's public log means signing events are recorded and verifiable; use it to detect unexpected signing and to audit your supply chain.
6. **Automate signing and verification** in the pipeline (sign at build, verify at deploy) so every artifact is signed and every deployment checks it — unsigned/wrongly-signed artifacts rejected.

### Cheatsheet

```
signing = origin + integrity (genuine + unmodified). historically blocked by KEY MANAGEMENT.
  Sigstore = KEYLESS signing -> makes it practical at scale.

keyless: authenticate with IDENTITY (OIDC — CI workload / developer) -> short-lived cert for signing
  -> NO key to store/protect/LEAK (the breakthrough)
Rekor: public tamper-evident TRANSPARENCY LOG (signing events auditable, can't secretly forge)

sign:   cosign sign <artifact>              (keyless, OIDC identity, immutable digest)
verify: cosign verify --certificate-identity=<expected> --certificate-oidc-issuer=<issuer> <artifact>
  -> check it was signed by the EXPECTED identity, not just "is signed"
  enforce at deploy (admission control) -> only trusted-identity artifacts run

prefer KEYLESS in CI (no key to inject/steal)
sign ATTESTATIONS too: SBOM (what) + provenance/SLSA (how) + scan results
automate sign (build) + verify (deploy)
```

### Reading the setup

- **Artifacts unsigned** = no way to verify origin or integrity; a tampered or substituted artifact deploys unchecked. Signing (now practical with keyless Sigstore) closes this — and the keyless model removes the old excuse of key-management pain.
- **Keyless signing in CI** = no signing key stored in the pipeline to steal, which removes the biggest historical signing risk. The identity-based model is both easier and more secure than managing keys.
- **Verification that only checks "is it signed", not the signer identity** = weak; an attacker who can sign with *any* trusted-by-your-config identity passes. Verify the *expected* identity (the specific CI workflow/team).
- **Signatures recorded in Rekor** = auditable, tamper-evident signing history; unexpected signing events are detectable. A supply-chain audit benefit.
- **Signing the artifact but not attestations** = you verify existence but not contents (SBOM) or build integrity (provenance); sign attestations too for full assurance.
- **Keyless signing with identity-checked verification enforced at deploy, attestations signed** = modern supply-chain integrity; only genuine, trusted-identity artifacts run.

### Pitfalls

- **Not signing because "key management is hard".** Keyless Sigstore removes that barrier — no key to manage or leak. The historical excuse no longer applies; sign.
- **Verifying "is signed" without checking identity.** An attacker signing with any identity your config trusts passes; verify the *expected* signer identity, not just the presence of a signature.
- **Managing long-lived keys in CI when keyless is available.** A key injected into the pipeline can be stolen; keyless tied to workload identity avoids this entirely.
- **Signing without deploy-time verification enforcement.** Signatures nobody checks protect nothing; enforce verification at admission/deploy.
- **Signing only the artifact.** Sign SBOMs and provenance attestations too, so you can verify contents and build integrity, not just existence.

### References

- Sigstore documentation (sigstore.dev), cosign, and Rekor transparency log
- The container supply-chain-for-images, devsecops artifact-integrity, and build-provenance skills
- The sbom-generation skill (signing SBOMs) and SLSA framework
- OpenSSF and Sigstore keyless-signing guides

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.