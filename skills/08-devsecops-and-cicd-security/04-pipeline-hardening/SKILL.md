---
name: pipeline-hardening
domain: 08-devsecops-and-cicd-security
description: Use when securing the CI/CD pipeline itself — least-privilege runners, pinned actions, protected branches, and the controls that stop the pipeline from becoming an attack path to production.
difficulty: intermediate
tags: [devsecops, cicd, pipeline, hardening, supply-chain]
tools: [github-actions, gitlab-ci]
---

## Purpose

The CI/CD pipeline has write access to production and runs whatever the latest commit tells it to — which makes it one of the most powerful and most attacked parts of the software supply chain. A compromised pipeline can inject malicious code into every build, steal secrets, and deploy to production. This skill covers hardening the pipeline itself so it doesn't become the attack path — securing the thing that secures everything else it touches.

## When to use it

Hardening any CI/CD setup, especially given how often pipelines are the target of supply-chain attacks. It's foundational: a pipeline that scans code but is itself compromised provides false assurance. Securing the pipeline is as important as what the pipeline does.

## Procedure

1. **Least-privilege the pipeline's access.** Pipelines often run with broad permissions (deploy to prod, read all secrets, write to any repo). Scope each pipeline/job to the minimum it needs — a build job doesn't need deploy credentials; a test job doesn't need production access. Over-privileged pipelines turn one compromised workflow into total compromise.
2. **Pin third-party actions/dependencies — a key supply-chain control.** Pipelines pull in reusable actions/steps (GitHub Actions, GitLab includes), and referencing them by mutable tag (`@v3`) means a compromised or repointed action runs in your pipeline. Pin third-party actions by **commit SHA**, not tag, so you run exactly the reviewed code. A tag can be moved to malicious code; a SHA can't.
3. **Protect the branches and the pipeline config.** Require reviews on the branches that trigger deployment, and protect the pipeline configuration itself (the workflow files) — if an attacker can edit the pipeline config in a PR, they can make it do anything. Require review for changes to pipeline definitions specifically.
4. **Secure secret handling.** Pipeline secrets (deploy keys, cloud credentials, signing keys) are high-value; scope them to the jobs that need them, don't expose them to PR builds from forks (a common exfiltration path — a malicious PR reads the secrets), and prefer short-lived/OIDC-based credentials over long-lived static secrets.
5. **Guard against poisoned pipeline execution.** An attacker who can influence what the pipeline runs (via a PR that modifies build scripts, or injects into a build step) can execute code in your trusted CI environment. Don't run untrusted PR code with access to secrets/production; separate untrusted-PR builds from privileged pipelines.
6. **Isolate and clean runners** (the secure-runners skill) — especially self-hosted runners, which persist state between jobs and can be a foothold.
7. **Log and monitor pipeline activity.** Treat the pipeline as the high-value target it is — log configuration changes, secret access, and deployments, and alert on anomalies (a workflow suddenly accessing secrets it never did).

## Cheatsheet

```
the pipeline has WRITE access to prod + runs whatever the latest commit says
  -> most powerful + most attacked part of the supply chain. secure the securer.

harden
  LEAST PRIVILEGE per job (build != deploy creds ; test != prod access)
    over-privileged pipeline = one workflow compromise -> total compromise
  PIN third-party actions by COMMIT SHA not tag (@v3 mutable -> repointed to malicious)
  PROTECT branches + the PIPELINE CONFIG itself (edit config in a PR = do anything)
    -> require review for workflow-definition changes
  SECRETS: scope to jobs, NOT exposed to fork-PR builds (malicious PR reads them),
    prefer short-lived / OIDC over static
  POISONED PIPELINE EXECUTION: don't run untrusted PR code WITH secrets/prod access
    -> separate untrusted-PR builds from privileged pipelines
  ISOLATE + clean runners (esp. self-hosted — persist state) [secure-runners]
  LOG + monitor (config changes, secret access, deploys) — it's a high-value target
```

## Reading the pipeline

- **Over-privileged pipeline jobs** (build jobs with deploy credentials, broad secret access) = one compromised workflow becomes total compromise; least-privilege per job is the core hardening. A common, high-impact finding.
- **Third-party actions referenced by mutable tag** (`@v3`) = a compromised or repointed action runs in your trusted pipeline — a real supply-chain vector. Pin by commit SHA.
- **Pipeline config editable via PR without review** = an attacker's PR can make the pipeline do anything (exfil secrets, inject code, deploy). Protect workflow definitions specifically.
- **Secrets exposed to fork-PR builds** = a malicious PR from a fork can read and exfiltrate your pipeline secrets; a classic attack. Don't expose secrets to untrusted PR builds.
- **Untrusted PR code running with secret/prod access** = poisoned pipeline execution; the attacker executes code in your trusted CI. Separate untrusted builds from privileged ones.
- **A hardened pipeline** (least privilege, SHA-pinned actions, protected config, scoped secrets, isolated runners, monitored) = the securer is itself secure; it can't be turned into the attack path.

## The fix / best practice

- **Least-privilege every job** to the minimum access it needs; separate build, test, and deploy privileges.
- **Pin third-party actions by commit SHA**, not tag, and review them before adopting.
- **Protect branches and pipeline configuration** with required review, especially the workflow files themselves.
- **Scope secrets to jobs**, keep them away from fork-PR builds, and prefer short-lived/OIDC credentials over static secrets.
- **Separate untrusted-PR execution** from privileged pipelines to prevent poisoned pipeline execution.
- **Isolate and clean runners**, and **log/monitor** the pipeline as the high-value target it is.

## Pitfalls

- **Over-privileged pipelines.** Broad access turns one compromised workflow into total compromise; scope every job to least privilege. The most common and impactful pipeline weakness.
- **Mutable action references.** `@v3` and tags can be repointed to malicious code; pin third-party actions by commit SHA.
- **Unprotected pipeline config.** If an attacker's PR can edit the workflow, they control the pipeline. Require review for pipeline-definition changes.
- **Secrets in fork-PR builds.** A malicious external PR reads and exfiltrates them; never expose secrets to untrusted PR builds.
- **Running untrusted code with privileges.** Poisoned pipeline execution lets an attacker run code in trusted CI; separate untrusted builds from privileged pipelines.
- **Trusting the pipeline blindly.** A pipeline that scans code but is itself compromised gives false assurance; harden and monitor the pipeline itself.

## References

- OWASP CI/CD Security Top 10 and SLSA framework
- GitHub Actions / GitLab CI security hardening documentation
- The secure-runners, artifact-integrity, and build-provenance-slsa skills
- The software-supply-chain-security domain
