# 06 — Cloud Security

The perimeter moved into IAM policies and resource configs. Most cloud incidents aren't clever exploits — they're a public bucket, an over-scoped role, or a key that leaked into a repo. This domain is about finding those before someone else does, across AWS, Azure, and GCP.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [s3-bucket-misconfiguration](01-s3-bucket-misconfiguration/SKILL.md) | Find and fix public/writable object storage | ✅ |
| 02 | [iam-privilege-escalation](02-iam-privilege-escalation/SKILL.md) | Trace paths from low priv to admin | ✅ |
| 03 | cloud-credential-hygiene | Rotate, scope, and detect leaked keys | TODO |
| 04 | security-group-review | Audit network exposure of cloud resources | TODO |
| 05 | [cloudtrail-and-audit-logging](05-cloudtrail-and-audit-logging/SKILL.md) | Turn on and read the audit trail that matters | ✅ |
| 06 | serverless-security | Lambda/Functions permissions and event injection | TODO |
| 07 | kms-and-secrets-management | Key policies, envelope encryption, secret stores | TODO |
| 08 | cspm-baseline | Continuous posture checks with open tooling | TODO |
| 09 | multi-account-guardrails | SCPs, landing zones, org-wide controls | TODO |
| 10 | [metadata-service-attacks](10-metadata-service-attacks/SKILL.md) | IMDS abuse via SSRF and the v2 fix | ✅ |

`s3-bucket-misconfiguration` is first because it's the classic cloud breach and the fastest win.
