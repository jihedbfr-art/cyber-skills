# 06 — Cloud Security

The perimeter moved into IAM policies and resource configs. Most cloud incidents aren't clever exploits — they're a public bucket, an over-scoped role, or a key that leaked into a repo. This domain is about finding those before someone else does, across AWS, Azure, and GCP.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [s3-bucket-misconfiguration](01-s3-bucket-misconfiguration/SKILL.md) | Find and fix public/writable object storage | ✅ |
| 02 | [iam-privilege-escalation](02-iam-privilege-escalation/SKILL.md) | Trace paths from low priv to admin | ✅ |
| 03 | [cloud-credential-hygiene](03-cloud-credential-hygiene/SKILL.md) | Rotate, scope, and detect leaked keys | ✅ |
| 04 | [security-group-review](04-security-group-review/SKILL.md) | Audit network exposure of cloud resources | ✅ |
| 05 | [cloudtrail-and-audit-logging](05-cloudtrail-and-audit-logging/SKILL.md) | Turn on and read the audit trail that matters | ✅ |
| 06 | [serverless-security](06-serverless-security/SKILL.md) | Lambda/Functions permissions and event injection | ✅ |
| 07 | [kms-and-secrets-management](07-kms-and-secrets-management/SKILL.md) | Key policies, envelope encryption, secret stores | ✅ |
| 08 | [cspm-baseline](08-cspm-baseline/SKILL.md) | Continuous posture checks with open tooling | ✅ |
| 09 | [multi-account-guardrails](09-multi-account-guardrails/SKILL.md) | SCPs, landing zones, org-wide controls | ✅ |
| 10 | [metadata-service-attacks](10-metadata-service-attacks/SKILL.md) | IMDS abuse via SSRF and the v2 fix | ✅ |

This domain is complete (10/10). `s3-bucket-misconfiguration` is first because it's the classic cloud breach and the fastest win.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>