---
format: "v2"
name: "kubernetes-rbac-audit"
title: "Kubernetes Rbac Audit"
title_fr: "Audit RBAC Kubernetes"
description: "Use when auditing Kubernetes RBAC for over-permissive roles and bindings — the excessive permissions that let a compromised workload or user take over the cluster."
description_fr: "À utiliser pour auditer le RBAC Kubernetes à la recherche de rôles et de bindings trop permissifs — les permissions excessives qui permettent à un workload compromis ou à un utilisateur de prendre le contrôle du cluster."
domain: "07-container-and-kubernetes-security"
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

Kubernetes RBAC controls who and what can do what in a cluster — and over-permissive roles are how a compromised pod or a stolen credential becomes cluster-wide compromise. A service account with too many rights, a role that grants `*` on everything, or the ability to create pods (and thus run arbitrary containers) can all lead to takeover. This skill covers auditing RBAC for the excessive permissions that create those escalation paths, the cluster equivalent of the cloud IAM-privesc review.

### When to use it

Auditing a Kubernetes cluster's security posture, after deployment, or when assessing whether a compromised workload could escalate. RBAC misconfiguration is a leading cause of Kubernetes compromise, making this a high-value audit.

### Procedure

1. **Enumerate roles, cluster roles, and their bindings.** Map who (users, groups, service accounts) has what permissions. `kubectl` and tools like `rbac-tool` or `kubectl-who-can` help visualise it:
   ```
   kubectl get clusterroles,roles,clusterrolebindings,rolebindings -A
   rbac-tool policy-rules -e '.*'          # who can do what
   kubectl who-can create pods             # who can perform a dangerous action
   ```
2. **Hunt wildcard and overly-broad permissions.** `verbs: ["*"]`, `resources: ["*"]`, or `apiGroups: ["*"]` grant far more than almost anything needs. `cluster-admin` bound to a workload service account is the worst case. These broad grants are the primary finding.
3. **Identify the escalation-enabling permissions specifically.** Certain permissions are dangerous because they lead to more:
   - **`create`/`update` on pods** — lets you run arbitrary containers (mount host paths, use privileged, access other secrets) — effectively code execution in the cluster.
   - **secrets access (`get`/`list`)** — read credentials, including other service accounts' tokens.
   - **`create` on rolebindings/clusterrolebindings** — grant yourself more permissions (RBAC escalation).
   - **`escalate`/`bind` verbs, `impersonate`, exec into pods, token creation** — all privilege-escalation primitives.
4. **Focus on service accounts, not just users.** Workload service accounts are what a compromised pod uses; an over-permissioned SA means a compromised pod inherits those rights. Default service accounts with excessive permissions are a common, dangerous pattern.
5. **Check for the default-token exposure.** Pods that mount a service-account token they don't need hand that token to any compromise; disable automounting where the workload doesn't call the API.
6. **Trace escalation paths.** Like cloud IAM, the risk is a *chain* — a pod's SA can read a secret that's a more privileged SA's token, or create a pod as a privileged SA. Map how a low-priv foothold could reach cluster-admin.
7. **Report the specific over-grants** and the escalation paths — the fix is tightening the role/binding, not a generic "improve RBAC".

### Cheatsheet

```
RBAC = who/what can do what in the cluster ; over-permission = pod/cred -> cluster takeover

enumerate: kubectl get clusterroles,roles,*bindings -A
  rbac-tool policy-rules | kubectl who-can <verb> <resource>

RED FLAGS
  verbs/resources/apiGroups = ["*"]  | cluster-admin bound to a workload SA

escalation-enabling permissions (dangerous -> lead to more)
  create/update PODS      -> run arbitrary containers (hostPath/privileged/other secrets)
  get/list SECRETS        -> read creds incl. other SA tokens
  create *ROLEBINDINGS    -> grant yourself more (RBAC escalation)
  escalate / bind / impersonate / pods/exec / token creation

focus on SERVICE ACCOUNTS (compromised pod inherits SA rights)
  default SA w/ excessive perms = common danger ; disable unused token automount
trace ESCALATION CHAINS (pod SA -> read a privileged SA token -> ...)
```

### Reading the audit

- **A workload service account bound to `cluster-admin` or with wildcard permissions** = a compromised pod using that SA owns the cluster; the worst and most impactful RBAC finding. Scope it to the minimum immediately.
- **`create`/`update` on pods granted broadly** = effectively cluster code execution; whoever has it can run privileged containers, mount host paths, and reach other secrets. A subtle but critical over-grant.
- **secrets `get`/`list` on a broad scope** = the SA can read credentials including other service accounts' tokens, enabling escalation to more privileged identities.
- **`create` on rolebindings, or `escalate`/`impersonate` verbs** = direct RBAC self-escalation — the SA can grant itself more. High severity.
- **An over-permissioned default service account** = every pod using it inherits the rights; a common, dangerous default that turns any pod compromise into more.
- **A traced chain from a low-priv SA to cluster-admin** = the real risk, expressed as the exact escalation path; that path is what to break.
- **Least-privilege roles scoped to specific verbs/resources/namespaces** = the good state; a compromised workload gains little.

### The fix

- **Least privilege on every role and binding** — grant only the specific verbs, resources, and namespaces a workload needs, never `*`. This is the direct remediation.
- **No `cluster-admin` for workloads** — reserve it for genuine cluster administration, never bind it to a service account.
- **Constrain the escalation-enabling permissions** (pod create, secrets, rolebinding create, impersonate/escalate) tightly, and keep them out of workload SAs.
- **Per-workload service accounts** with minimal rights, not shared or default SAs; disable token automounting for pods that don't need the API.
- **Namespace-scope roles** where possible rather than cluster-wide, to limit blast radius.
- **Audit continuously** — RBAC drifts as teams add permissions; re-audit regularly and use admission control (that skill) to prevent dangerous grants.

### Pitfalls

- **Wildcard permissions.** `*` on verbs/resources/apiGroups grants far more than needed and is the root of most RBAC over-permission. Scope explicitly.
- **cluster-admin on workloads.** Binding it to a service account means any pod compromise is cluster compromise. Never do it for workloads.
- **Overlooking escalation-enabling verbs.** pod create, secrets access, rolebinding create, and impersonate/escalate lead to more privilege even without obvious admin rights. Audit for these specifically.
- **Ignoring service accounts.** Users get attention but workload SAs are what compromised pods use; over-permissioned SAs (especially defaults) are the real risk.
- **Auditing once.** RBAC accumulates permissions over time; re-audit and enforce with admission control.

### References

- Kubernetes RBAC documentation and CIS Kubernetes Benchmark
- rbac-tool, kubectl-who-can, and Kubernetes RBAC audit tooling
- The cloud iam-privilege-escalation skill (same reasoning), pod-security-standards, and admission-control skills
- MITRE ATT&CK for Containers

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.