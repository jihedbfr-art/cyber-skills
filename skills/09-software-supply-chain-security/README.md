# 09 — Software Supply Chain Security

Your code is a small fraction of what you ship. The rest is dependencies, base images, and build tools you didn't write and mostly don't read. This domain is about knowing what's in the box and proving nobody swapped it.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [sbom-generation](01-sbom-generation/SKILL.md) | Produce and read a software bill of materials | ✅ |
| 02 | [dependency-confusion](02-dependency-confusion/SKILL.md) | Understand and block the namespace attack | ✅ |
| 03 | [typosquat-detection](03-typosquat-detection/SKILL.md) | Catch malicious lookalike packages | ✅ |
| 04 | [artifact-signing-sigstore](04-artifact-signing-sigstore/SKILL.md) | Sign and verify with cosign/sigstore | ✅ |
| 05 | [lockfile-integrity](05-lockfile-integrity/SKILL.md) | Pin and verify transitive dependencies | ✅ |
| 06 | [vulnerable-dependency-triage](06-vulnerable-dependency-triage/SKILL.md) | Tell exploitable from merely-flagged | ✅ |
| 07 | [build-provenance](07-build-provenance/SKILL.md) | Attest the build environment and inputs | ✅ |
| 08 | [third-party-risk](08-third-party-risk/SKILL.md) | Assess a vendor before you depend on them | ✅ |
| 09 | [package-repo-hardening](09-package-repo-hardening/SKILL.md) | Lock down internal registries | ✅ |
| 10 | [malicious-package-response](10-malicious-package-response/SKILL.md) | What to do when a dep goes bad | ✅ |

This domain is complete (10/10). Start with `sbom-generation` — you can't defend a supply chain you can't enumerate.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>