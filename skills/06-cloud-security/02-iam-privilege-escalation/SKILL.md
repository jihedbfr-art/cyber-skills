---
format: "v2"
name: "iam-privilege-escalation"
title: "Iam Privilege Escalation"
title_fr: "Élévation de privilèges IAM"
description: "Use when you have limited cloud credentials and need to check whether IAM misconfigurations let you reach higher privileges — mapping escalation paths and closing them."
description_fr: "À utiliser quand on dispose d'identifiants cloud limités et qu'il faut vérifier si des mauvaises configurations IAM permettent d'atteindre des privilèges plus élevés — pour cartographier les chemins d'escalade et les refermer."
domain: "06-cloud-security"
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

In the cloud, privilege escalation is rarely an exploit — it's a policy that grants one permission too many. `iam:PassRole` plus `lambda:CreateFunction`, or `iam:CreatePolicyVersion` on your own role, and a read-only identity becomes admin. This skill covers enumerating what a set of credentials can do and finding the path up, then removing it.

Scoped to AWS here because it's the most common; the reasoning (find a permission that grants more permissions) transfers to Azure and GCP.

### When to use it

Authorised cloud assessments, or auditing your own account for latent escalation paths. Assume-breach reviews especially: "if this CI role leaked, could it become admin?"

Only against accounts you own or are authorised to test. Enumerating IAM on someone else's account is unauthorised access.

### Procedure

1. Establish what identity you're operating as and confirm scope:
   ```
   aws sts get-caller-identity
   ```
2. Enumerate the permissions attached to the identity. If you can read IAM, list the policies directly; if not, brute-force which API calls succeed:
   ```
   aws iam list-attached-user-policies --user-name <u>
   aws iam get-user-policy --user-name <u> --policy-name <p>
   # if IAM read is denied, probe what's allowed:
   enumerate-iam --access-key ... --secret-key ...
   ```
3. Map the escalation paths automatically — this is the part you don't do by eye at scale:
   ```
   pmapper graph create
   pmapper query "preset privesc <principal>"
   ```
4. Reason about the high-signal permission combinations by hand as a cross-check. Classic AWS paths include:
   - `iam:CreatePolicyVersion` — rewrite a policy you're attached to, granting yourself `*`.
   - `iam:PassRole` + `lambda:CreateFunction`/`ec2:RunInstances` — run code as a more privileged role.
   - `iam:AttachUserPolicy` — attach `AdministratorAccess` to yourself.
   - `iam:CreateAccessKey` on another user — mint credentials for a bigger identity.
   - `sts:AssumeRole` on an over-trusting role.
5. Validate a path carefully in an authorised test — demonstrate reachability (e.g. that `PassRole` accepts an admin role) without actually leaving persistent admin access behind. Clean up anything you create.

### Cheatsheet

```bash
aws sts get-caller-identity
aws iam list-attached-user-policies --user-name U
aws iam list-user-policies --user-name U
aws iam get-policy-version --policy-arn ARN --version-id v1

enumerate-iam --access-key AK --secret-key SK    # what can these keys do?
pmapper graph create && pmapper query "preset privesc U"   # escalation paths

iam:CreatePolicyVersion   iam:SetDefaultPolicyVersion
iam:AttachUserPolicy      iam:AttachRolePolicy      iam:PutUserPolicy
iam:PassRole              iam:CreateAccessKey       sts:AssumeRole
iam:UpdateAssumeRolePolicy   lambda:CreateFunction   ec2:RunInstances
```

### Reading the output

- **`pmapper` returning a path to an admin node** is your finding, with the exact edges (which permission enables which hop) — that's what goes in the report.
- **A wildcard `Action: "*"` or `iam:*`** on a non-admin principal is an obvious over-grant.
- **`iam:PassRole` with `Resource: "*"`** is a red flag — it lets the identity hand *any* role to a service it can invoke.
- **A role trust policy with a broad `Principal`** (a whole account, or `*`) means anyone in that scope can assume it.

### The fix

Least privilege, and remove the permissions that grant permissions:

- Strip wildcard actions and resources. Grant the specific API calls a role needs, scoped to specific ARNs.
- Constrain `iam:PassRole` to the exact roles a service legitimately needs, never `Resource: "*"`.
- Keep the permission-granting IAM actions (`Create*Policy*`, `Attach*Policy`, `PutUserPolicy`, `CreateAccessKey`, `UpdateAssumeRolePolicy`) inside a small, monitored admin boundary — use **permission boundaries** and SCPs to cap what any principal can reach even if its policy says otherwise.
- Tighten role trust policies to specific principals with conditions (`aws:SourceArn`, external ID).
- Alert on the escalation-enabling calls in CloudTrail so an attempt is visible.

### Pitfalls

- **Reading policies without evaluating boundaries.** An allow can be capped by a permission boundary or SCP — check the effective permissions, not just the attached policy.
- **Leaving admin access behind after a test.** If you demonstrate a path, revert it. Document instead of persist.
- **Chasing single permissions.** Escalation usually needs a *combination*; that's why graph tooling beats eyeballing.
- **Ignoring roles you can assume.** The path often runs through `AssumeRole` into a more privileged identity, not a direct policy edit.

### References

- AWS IAM best practices documentation
- Principal Mapper (pmapper) documentation
- Rhino Security Labs — AWS IAM privilege escalation methods
- CWE-269 (Improper Privilege Management)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.