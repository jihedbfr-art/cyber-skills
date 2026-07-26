# 08 — DevSecOps & CI/CD Security

The pipeline has write access to production and runs whatever the last commit told it to. That makes it a target and a control point at the same time. This domain covers securing the build and using it to enforce security on everything that passes through.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | secrets-scanning-in-pipelines | Block commits and builds that leak credentials | TODO |
| 02 | sast-integration | Wire static analysis into the build without drowning in noise | TODO |
| 03 | dependency-scanning | Fail builds on known-vulnerable packages | TODO |
| 04 | pipeline-hardening | Least-privilege runners, pinned actions, protected branches | TODO |
| 05 | dast-in-cicd | Run dynamic scans against ephemeral environments | TODO |
| 06 | iac-scanning | Catch misconfig in Terraform/CloudFormation pre-deploy | TODO |
| 07 | artifact-integrity | Sign build outputs, verify before deploy | TODO |
| 08 | secure-runners | Isolate and clean self-hosted runners | TODO |
| 09 | policy-as-code | Codify the gates so they can't be skipped | TODO |
| 10 | build-provenance-slsa | Prove where an artifact came from | TODO |

TODO: no skills written yet. `secrets-scanning-in-pipelines` is the highest-value starting point.
