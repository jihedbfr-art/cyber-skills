---
name: cloud-credential-hygiene
domain: 06-cloud-security
description: Use when reviewing how cloud credentials are created, stored, scoped, and rotated — stopping the long-lived leaked key that's behind most cloud breaches.
difficulty: beginner
tags: [cloud, aws, credentials, iam, keys, rotation]
tools: [aws-cli, git-secrets]
---

## Purpose

Most cloud compromises don't start with an exploit — they start with a credential. A long-lived access key committed to a repo, an over-scoped key that never expires, a root account with programmatic keys. This skill covers the practices that keep credentials short-lived, tightly scoped, and out of the places attackers look.

## When to use it

Auditing a cloud account's security posture, onboarding a new one, or after any leak scare. It pairs with the S3 and IAM-privesc skills — hygiene is the layer that stops the initial foothold those attacks build on.

## Procedure

1. **Find long-lived static keys and question each one.** IAM users with access keys are the main leak risk. List them and their age — old keys are both more likely to have leaked and more likely forgotten:
   ```
   aws iam list-users
   aws iam list-access-keys --user-name <u>
   aws iam get-access-key-last-used --access-key-id <id>
   ```
2. **Eliminate root-account keys.** The root account should have **no** access keys at all — if any exist, that's a top finding. Root is for a handful of console tasks, never programmatic use.
3. **Prefer short-lived credentials over static keys.** For workloads, use instance/pod roles (IAM roles) that vend temporary credentials automatically — no static key to leak. For humans, use SSO/identity-center federation, not per-user access keys.
4. **Scope every credential to least privilege** (ties into iam-privilege-escalation) — a leaked key that can do little is a contained incident, not a breach.
5. **Rotate what must stay static**, and enforce a max age. Track last-used so you can delete keys nothing uses.
6. **Keep keys out of code.** Confirm secret scanning is in place (the DevSecOps skill) and that credentials come from a secret manager or role, never from a committed file or a baked-in image.
7. **Detect and alert** on new key creation, root usage, and credentials appearing in public repos (the OSINT github-secret-recon skill from the defender's side).

## Cheatsheet

```bash
# find static keys + age + usage
aws iam list-users --query 'Users[].UserName'
aws iam list-access-keys --user-name U
aws iam get-access-key-last-used --access-key-id AKIA...

# the top findings to hunt
- root account with ANY access key            -> critical, remove
- access keys older than your rotation policy  -> rotate/remove
- keys with last-used = never                  -> delete (unused)
- IAM users with keys where a role would do    -> migrate to roles/SSO

# credential source, in order of preference
IAM roles (workloads) > SSO/federation (humans) > rotated scoped keys > static keys (avoid)
```

## Reading the review

- **Root account access keys** = the single worst finding here; root credentials leaking is game over. Remove them.
- **Old, never-rotated static keys** = high leak probability and often forgotten — each is a latent breach. Rotate or delete.
- **Keys with `last-used: never`** = unused credentials that can only hurt; delete them.
- **Workloads using static keys instead of roles** = an unnecessary long-lived secret where temporary credentials were available. Migrate.
- **A key that's broadly scoped** turns a leak into a full compromise; narrow it (least privilege) so exposure is survivable.

## The fix / best practice

- **No root keys, ever.** Lock the root account behind MFA and use it only for the rare tasks that require it.
- **Roles for workloads, SSO for humans** — vend temporary credentials so there's no static key to leak. This removes the most common breach vector outright.
- **Least-privilege scoping** on everything, so a leaked credential is contained.
- **Rotate and expire** any static keys that must exist; enforce a max age and delete unused keys.
- **Secret scanning + secret manager** so credentials never enter code or images.
- **Alerting** on key creation, root activity, and public-repo leaks — detection backs up the prevention.

## Pitfalls

- **Root programmatic keys.** Convenient, catastrophic if leaked. There's essentially no legitimate reason for them.
- **Static keys where a role would work.** Every avoidable long-lived secret is avoidable risk.
- **Rotating without scoping.** A freshly rotated key that can still do everything is still a full-breach risk if it leaks.
- **Forgetting unused keys.** A key nobody uses is a key nobody's watching — delete it.

## References

- AWS IAM best practices (root account, temporary credentials, rotation)
- CIS AWS Foundations Benchmark (credential-related controls)
- AWS IAM Identity Center / SSO documentation
- CWE-798 (Use of Hard-coded Credentials)
