---
format: "v2"
name: "iac-scanning"
title: "Iac Scanning"
title_fr: "Analyse de l'infrastructure as code"
description: "Use when scanning infrastructure-as-code for misconfigurations before deploy — catching the insecure Terraform/CloudFormation/Kubernetes settings that become cloud breaches."
description_fr: "À utiliser pour scanner l'infrastructure as code à la recherche de mauvaises configurations avant déploiement — afin d'intercepter les réglages Terraform/CloudFormation/Kubernetes non sécurisés qui se transforment en brèches cloud."
domain: "08-devsecops-and-cicd-security"
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

Infrastructure is code now — Terraform, CloudFormation, Kubernetes manifests — and an insecure setting there (a public S3 bucket, an open security group, an unencrypted database) becomes a real cloud misconfiguration the moment it deploys. IaC scanning catches those misconfigurations *before* deployment, at the code stage, where they're a one-line fix instead of a production exposure. This skill covers integrating IaC scanning into the pipeline, shifting cloud security left.

### When to use it

Any pipeline that deploys infrastructure via IaC (most modern ones). It's the shift-left counterpart to the cloud-security domain's posture management (CSPM) — IaC scanning prevents misconfigurations at the code stage, CSPM catches them at runtime; together they cover the lifecycle.

### Procedure

1. **Scan IaC in the pipeline before deploy.** Wire an IaC scanner (Checkov, tfsec, Trivy, KICS) into CI to check Terraform/CloudFormation/Kubernetes/ARM against misconfiguration rules on every change. Catching the public bucket in the pull request is far cheaper than catching it in production.
2. **Cover the high-impact misconfiguration classes** — the same ones the cloud domain warns about, but caught in code:
   - Public storage (S3 buckets, blob containers open to the world).
   - Open security groups / firewall rules (`0.0.0.0/0` on sensitive ports).
   - Unencrypted data (storage, databases without encryption-at-rest).
   - Over-permissive IAM (wildcard policies).
   - Missing logging/audit configuration, public databases, weak TLS.
3. **Map findings to real risk.** IaC scanners flag against a broad policy set, some of which won't matter for your context; prioritise the genuinely dangerous misconfigurations (public exposure, over-permission) over stylistic or low-impact rules — the same tuning discipline as SAST.
4. **Gate on high-severity misconfigurations.** Fail the pipeline on the dangerous ones (public exposure, open sensitive ports, wildcard IAM) so they can't deploy; report the rest. This prevents the misconfiguration from ever reaching the cloud.
5. **Scan modules and dependencies too.** IaC pulls in reusable modules (Terraform registry); a misconfigured or malicious module is your exposure. Scan what you pull in, not just what you wrote.
6. **Handle the existing-infrastructure baseline.** As with SAST, turning IaC scanning on surfaces many pre-existing findings; baseline them and gate on new misconfigurations so the pipeline stays usable while debt is worked down.
7. **Close the loop with CSPM.** IaC scanning prevents misconfigurations pre-deploy, but drift happens (manual changes in the console). Pair with runtime CSPM (cloud domain) so a misconfiguration introduced outside IaC is still caught. Fix drift back in the IaC, not the console.

### Cheatsheet

```
infra = code (Terraform/CloudFormation/K8s) ; insecure setting -> real cloud misconfig on deploy
  IaC scanning = catch it BEFORE deploy (one-line fix in PR vs production exposure)
  shift-left counterpart to CSPM (runtime)

integrate
  SCAN in CI before deploy (Checkov / tfsec / Trivy / KICS) on every change
  cover high-impact classes (same as cloud domain, caught in code)
    public storage | open security groups (0.0.0.0/0 sensitive ports) | unencrypted data
    | wildcard IAM | missing logging | public DBs | weak TLS
  MAP to real risk (broad rule sets -> prioritise dangerous over stylistic — SAST-style tuning)
  GATE on high-severity (public exposure, open ports, wildcard IAM) -> can't deploy ; report rest
  scan MODULES/dependencies too (Terraform registry — misconfigured module = your exposure)
  EXISTING infra: baseline, gate on NEW (keep pipeline usable)
  pair with CSPM (drift happens — console changes) ; fix drift in the IaC not the console
```

### Reading the results

- **A public storage bucket or open security group flagged in the IaC** = a cloud breach caught at the code stage — a one-line fix now versus a production exposure and possible incident later. The highest-value IaC-scanning catch, and exactly what shift-left is for.
- **Wildcard IAM policies in the IaC** = over-permission caught before it deploys; the cloud IAM-privesc paths prevented at source.
- **Findings against a broad policy set, many low-impact** = needs the same tuning as SAST; prioritise the genuinely dangerous misconfigurations over stylistic rules, or fatigue sets in.
- **A misconfigured reused module** = your exposure even though you didn't write it; scan modules and dependencies, not just your own code.
- **Turning IaC scanning on and blocking all deploys on pre-existing findings** = an unusable pipeline; baseline existing infra and gate on new misconfigurations.
- **A misconfiguration introduced in the console (drift)** = IaC scanning won't catch it (it's not in the code); CSPM does. Pair them, and fix drift back in the IaC.
- **Pre-deploy IaC scanning gating dangerous misconfigs, paired with runtime CSPM** = cloud security covered across the lifecycle.

### Pitfalls

- **Only catching misconfigurations at runtime (CSPM), not in code.** IaC scanning prevents them pre-deploy, far cheaper than remediating a live exposure. Shift the check left.
- **Not tuning the broad rule set.** IaC scanners flag many low-impact rules; without prioritising the dangerous ones, fatigue sets in. Tune like SAST.
- **Ignoring modules/dependencies.** A misconfigured reused module is your exposure; scan what you pull in.
- **Blocking on existing debt.** Turning scanning on and gating all deploys on pre-existing findings makes the pipeline unusable; baseline and gate on new.
- **Fixing drift in the console.** Manual console fixes get overwritten by the next IaC apply; fix misconfigurations in the IaC so they stick.
- **Treating IaC scanning as sufficient alone.** Drift and console changes bypass it; pair with runtime CSPM.

### References

- Checkov, tfsec, Trivy, KICS documentation
- The cloud-security domain (the misconfigurations, and cspm-baseline for runtime)
- The sast-integration and policy-as-code skills (same integration discipline)
- CIS benchmarks for cloud providers

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.