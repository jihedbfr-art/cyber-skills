# 08 — DevSecOps & CI/CD Security

The pipeline has write access to production and runs whatever the last commit told it to. That makes it a target and a control point at the same time. This domain covers securing the build and using it to enforce security on everything that passes through.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [secrets-scanning-in-pipelines](01-secrets-scanning-in-pipelines/SKILL.md) | Block commits and builds that leak credentials | ✅ |
| 02 | [sast-integration](02-sast-integration/SKILL.md) | Wire static analysis into the build without drowning in noise | ✅ |
| 03 | [dependency-scanning](03-dependency-scanning/SKILL.md) | Fail builds on known-vulnerable packages | ✅ |
| 04 | [pipeline-hardening](04-pipeline-hardening/SKILL.md) | Least-privilege runners, pinned actions, protected branches | ✅ |
| 05 | [dast-in-cicd](05-dast-in-cicd/SKILL.md) | Run dynamic scans against ephemeral environments | ✅ |
| 06 | [iac-scanning](06-iac-scanning/SKILL.md) | Catch misconfig in Terraform/CloudFormation pre-deploy | ✅ |
| 07 | [artifact-integrity](07-artifact-integrity/SKILL.md) | Sign build outputs, verify before deploy | ✅ |
| 08 | [secure-runners](08-secure-runners/SKILL.md) | Isolate and clean self-hosted runners | ✅ |
| 09 | [policy-as-code](09-policy-as-code/SKILL.md) | Codify the gates so they can't be skipped | ✅ |
| 10 | [build-provenance-slsa](10-build-provenance-slsa/SKILL.md) | Prove where an artifact came from | ✅ |

This domain is complete (10/10). `secrets-scanning-in-pipelines` is the highest-value starting point; `policy-as-code` ties the gates into unskippable enforcement.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>