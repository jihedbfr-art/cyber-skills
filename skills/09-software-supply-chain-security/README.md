# 09 — Software Supply Chain Security

Your code is a small fraction of what you ship. The rest is dependencies, base images, and build tools you didn't write and mostly don't read. This domain is about knowing what's in the box and proving nobody swapped it.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [sbom-generation](01-sbom-generation/SKILL.md) | Produce and read a software bill of materials | ✅ |
| 02 | dependency-confusion | Understand and block the namespace attack | TODO |
| 03 | typosquat-detection | Catch malicious lookalike packages | TODO |
| 04 | artifact-signing-sigstore | Sign and verify with cosign/sigstore | TODO |
| 05 | lockfile-integrity | Pin and verify transitive dependencies | TODO |
| 06 | vulnerable-dependency-triage | Tell exploitable from merely-flagged | TODO |
| 07 | build-provenance | Attest the build environment and inputs | TODO |
| 08 | third-party-risk | Assess a vendor before you depend on them | TODO |
| 09 | package-repo-hardening | Lock down internal registries | TODO |
| 10 | malicious-package-response | What to do when a dep goes bad | TODO |

Start with `sbom-generation` (done) — you can't defend a supply chain you can't enumerate.
