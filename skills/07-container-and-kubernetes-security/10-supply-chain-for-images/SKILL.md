---
name: supply-chain-for-images
domain: 07-container-and-kubernetes-security
description: Use when securing the container image supply chain — signing and verifying images so only trusted, unmodified images run, closing the gap between building and deploying.
difficulty: intermediate
tags: [containers, supply-chain, signing, cosign, provenance]
tools: [cosign, sigstore, kyverno]
---

## Purpose

Between building an image and running it in production, a lot can go wrong: a malicious image pushed to your registry under a trusted name, a tampered image, or simply an unvetted image from an untrusted source deployed by mistake. Container image supply-chain security ensures only trusted, unmodified images run — through signing and verification. This skill covers signing images and enforcing that only signed, trusted images deploy, closing the build-to-deploy gap. It's the container-specific application of the software supply-chain domain.

## When to use it

Hardening the path from image build to production deployment, especially where images come from multiple sources or the registry is a trust boundary. It pairs with admission control (which enforces the verification) and image scanning (which vets contents).

## Procedure

1. **Understand the trust gap.** An image tag (`myapp:latest`) is just a pointer; without verification, you're trusting that whatever's behind that tag is what you think — but a compromised registry, a typosquatted image, or a supply-chain attack can put a malicious image there. Signing and verification close this gap by proving an image's origin and integrity.
2. **Sign your images.** Use Sigstore/cosign to cryptographically sign images at build time, so their origin and integrity can be verified. Signing produces a signature tied to your identity/key that verification checks against:
   ```
   cosign sign <registry>/myapp@sha256:<digest>
   ```
   Sign the immutable digest, not a mutable tag.
3. **Verify signatures before deployment — the enforcement.** Configure admission control (Kyverno/OPA, the admission-control skill) to reject any image that isn't signed by a trusted key. Now only images you signed (or that come from trusted, signed sources) can run; an unsigned or tampered image is blocked at deploy:
   ```
   cosign verify <image>            # verify manually
   # + admission policy requiring valid signatures cluster-wide
   ```
4. **Pin images by digest, not tag.** Tags are mutable — `latest` (or even a version tag) can be repointed to a different image. Deploying by immutable digest (`@sha256:...`) ensures you run exactly the image you vetted and signed. Mutable-tag deployment undermines the whole chain.
5. **Restrict registries.** Allow images only from approved registries (enforced via admission control), so an image from an untrusted or public registry can't be deployed. Combine with signing — trusted source *and* valid signature.
6. **Generate and check provenance/SBOM.** Attach build provenance (how and where the image was built — SLSA) and an SBOM (what's inside — the supply-chain domain) so you can verify the build's integrity and inventory its contents, and check both at admission where possible.
7. **Automate the whole chain in CI/CD.** Build → scan → sign → push → (at deploy) verify. Each step automated and enforced so a human can't skip it. This is the container application of the DevSecOps pipeline discipline.

## Cheatsheet

```
build-to-deploy trust gap: a TAG is just a pointer -> compromised registry / typosquat /
  supply-chain attack can put a malicious image behind a trusted name.
  signing + verification close it (prove ORIGIN + INTEGRITY).

1. SIGN images (Sigstore/cosign) at build — sign the immutable DIGEST not a tag
     cosign sign registry/app@sha256:<digest>
2. VERIFY before deploy (enforcement): admission control (Kyverno/OPA) rejects unsigned
     -> only images signed by a trusted key can run
3. PIN by DIGEST not tag (tags mutable — `latest`/version can be repointed)
     -> run exactly the image you vetted+signed
4. RESTRICT registries (approved only, via admission) — trusted source AND valid signature
5. PROVENANCE (SLSA) + SBOM: how/where built + what's inside -> verify integrity + inventory
6. AUTOMATE in CI/CD: build -> scan -> sign -> push -> (deploy) verify. enforced, unskippable.
```

## Reading the chain

- **Deploying images with no signature verification** = you're trusting the tag blindly; a compromised registry or a malicious image pushed under a trusted name runs unchecked. Signing plus admission-enforced verification closes this — a high-value control.
- **An admission policy requiring signed images from approved registries** = only trusted, verified images can run; an unsigned, tampered, or untrusted-source image is blocked at deploy. The enforcement that makes signing meaningful.
- **Deploying by mutable tag** (`latest` or a version tag) = the tag can be repointed to a different image after you vetted it; you may run something other than what you signed. Pin by digest.
- **Signing without verification enforcement** = signatures nobody checks protect nothing; the value is in admission rejecting unsigned images. Signing and verification must go together.
- **Provenance and SBOM attached and checked** = you can verify how the image was built and what's inside, catching build tampering and known-vulnerable contents.
- **A fully automated build→scan→sign→push→verify pipeline** = the strong state; only trusted, signed, vetted images reach production, and no step can be skipped.

## The fix / best practice

- **Sign images at build** with Sigstore/cosign, signing the immutable digest.
- **Enforce signature verification at admission** — reject unsigned images cluster-wide (admission-control skill), so signing actually gates deployment.
- **Deploy by digest, not tag**, to run exactly what you vetted and signed.
- **Restrict to approved registries** and require both trusted source and valid signature.
- **Attach and check provenance (SLSA) and SBOM** for build integrity and content inventory.
- **Automate the whole chain in CI/CD** so build, scan, sign, and verify are enforced and unskippable.

## Pitfalls

- **No verification at deploy.** Signing without admission-enforced verification is pointless — signatures nobody checks protect nothing. Enforce verification.
- **Trusting tags.** Mutable tags can be repointed to malicious images after vetting; deploy by immutable digest.
- **Verifying source but not signature (or vice versa).** A trusted registry can still be compromised; require both an approved source and a valid signature.
- **Manual, skippable steps.** If signing/verification isn't automated and enforced in the pipeline, someone skips it. Bake it into CI/CD as a hard gate.
- **Ignoring provenance/SBOM.** Signing proves origin but not that the build wasn't tampered or that contents are safe; add provenance and SBOM checks.

## References

- Sigstore / cosign documentation and the SLSA framework
- The software-supply-chain-security domain (sbom-generation, artifact-signing) — the general case
- The admission-control and container-image-scanning skills
- Kyverno/OPA image verification policies; NIST SP 800-190
