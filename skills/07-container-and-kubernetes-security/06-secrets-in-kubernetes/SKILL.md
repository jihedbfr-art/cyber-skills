---
format: "v2"
name: "secrets-in-kubernetes"
title: "Secrets In Kubernetes"
title_fr: "Secrets dans Kubernetes"
description: "Use when handling secrets in Kubernetes — stopping the plaintext, over-exposed secrets that leak credentials, and using proper secret management with encryption and access control."
description_fr: "À utiliser pour gérer les secrets dans Kubernetes — éviter les secrets en clair et trop exposés qui font fuiter des identifiants, et mettre en place une vraie gestion des secrets avec chiffrement et contrôle d'accès."
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

Kubernetes Secrets look like a secure place for credentials, but by default they're only base64-encoded (not encrypted), stored in etcd, and readable by anyone with the right RBAC — so they're easy to expose. This skill covers handling secrets in Kubernetes properly: not committing them in plaintext, encrypting them at rest, restricting access, and integrating a real secret manager, so a credential in the cluster doesn't become a credential in an attacker's hands.

### When to use it

Deploying anything that needs credentials in Kubernetes (almost everything), and auditing how secrets are handled. It's a common weak point because the default Secret object *feels* secure but isn't, and it pairs with the RBAC and supply-chain skills.

### Procedure

1. **Understand what Kubernetes Secrets are — and aren't.** A Secret is base64-encoded, **not encrypted** by default; it's stored in etcd and accessible via the API to anyone with `get`/`list` on secrets in that namespace. Base64 is encoding, not security — treat a Secret's contents as readable by anyone with API access to it.
2. **Never commit secrets in plaintext.** The most common leak: secrets hardcoded in manifests, Helm values, or committed to git (the secret-scanning skill). Keep plaintext secrets out of repos entirely.
3. **Encrypt secrets at rest.** Enable etcd encryption-at-rest (encryption providers) so secrets aren't stored in plaintext in etcd — otherwise anyone with etcd access (or an etcd backup) reads them all. This is a cluster-level control worth confirming is on.
4. **Restrict access with RBAC.** Limit which service accounts and users can `get`/`list` secrets, and scope to specific secrets/namespaces (the RBAC skill). A workload that can read all secrets in a namespace is an escalation path; grant access to only the secrets a workload needs.
5. **Use a real secret manager for production.** Integrate an external secret store (HashiCorp Vault, cloud secret managers) via the External Secrets Operator or CSI driver, so secrets live in a purpose-built system with rotation, audit, and fine-grained access — and are injected into pods at runtime rather than stored as static Kubernetes Secrets.
6. **For GitOps, use encrypted secrets.** If secrets must live in git (GitOps), encrypt them (Sealed Secrets, SOPS) so the repo holds only ciphertext that only the cluster can decrypt — never plaintext.
7. **Minimise secret exposure in pods.** Mount only the secrets a pod needs, prefer file mounts over environment variables (env vars leak more easily — into logs, child processes, crash dumps), and disable service-account token automounting where unused.

### Cheatsheet

```
K8s Secret LOOKS secure but by default: base64 (NOT encrypted) + in etcd + API-readable via RBAC
  base64 = encoding, not security. treat contents as readable by anyone with API access.

do
  NEVER commit plaintext secrets (manifests/Helm/git) — #1 leak (secret-scanning)
  ENCRYPT at rest: enable etcd encryption-at-rest (else etcd/backup = all secrets)
  RBAC: restrict get/list on secrets to specific SAs + specific secrets/namespaces
    (workload reading ALL namespace secrets = escalation path)
  REAL SECRET MANAGER for prod: Vault / cloud manager via External Secrets Operator / CSI
    -> rotation, audit, fine-grained, runtime injection (not static K8s Secrets)
  GitOps -> ENCRYPTED secrets (Sealed Secrets / SOPS): repo holds ciphertext only
  pods: mount only needed secrets ; FILE mounts > env vars (env leaks to logs/children/dumps)
    ; disable unused SA token automount
```

### Reading the handling

- **Secrets committed in plaintext** (in manifests, Helm values, or git) = a direct credential leak, readable by anyone with repo access; the most common Kubernetes secret failure. Get them out of the repo and rotate.
- **etcd encryption-at-rest not enabled** = all secrets are plaintext in etcd; anyone with etcd access or a backup reads every secret in the cluster. A high-impact cluster-level gap.
- **A workload with `get`/`list` on all secrets in a namespace** = it can read credentials it shouldn't, including other workloads' — an escalation path (ties into RBAC). Scope to only its own secrets.
- **Static Kubernetes Secrets for production credentials** = no rotation, coarse access, and the base64/etcd exposure; a real secret manager (Vault/cloud) with runtime injection is far stronger for production.
- **Secrets in environment variables** = they leak more readily (into logs, child processes, crash dumps, `kubectl describe`); file mounts are safer. Prefer mounts.
- **Encrypted-at-rest secrets, RBAC-restricted, from a secret manager, mounted minimally** = the strong state; a cluster compromise yields far less.

### The fix / best practice

- **Keep plaintext secrets out of repos** entirely; scan for them (secret-scanning skill).
- **Enable etcd encryption-at-rest** so secrets aren't plaintext in the datastore or backups.
- **Restrict secret access via RBAC** to specific service accounts and specific secrets/namespaces.
- **Use a real secret manager** (Vault, cloud) via External Secrets Operator / CSI for production — rotation, audit, and runtime injection instead of static Secrets.
- **Encrypt GitOps secrets** with Sealed Secrets or SOPS so repos hold only ciphertext.
- **Minimise pod exposure** — mount only needed secrets, prefer files over env vars, disable unused token automounting.

### Pitfalls

- **Treating base64 as security.** Kubernetes Secrets are encoded, not encrypted; their contents are readable by anyone with API access. Don't rely on the Secret object alone.
- **Committing plaintext secrets.** The most common leak — in manifests, Helm values, or git. Keep them out and scan.
- **No encryption-at-rest.** Without it, etcd (and its backups) holds every secret in plaintext; one etcd access reads them all. Enable it.
- **Over-broad secret RBAC.** A workload able to read all namespace secrets is an escalation path. Scope tightly.
- **Static Secrets for production.** No rotation or fine-grained control; use a real secret manager with runtime injection.
- **Secrets in env vars.** They leak into logs, child processes, and crash dumps; prefer file mounts.

### References

- Kubernetes Secrets and encryption-at-rest documentation
- HashiCorp Vault, External Secrets Operator, Secrets Store CSI Driver
- Sealed Secrets and SOPS (GitOps secret encryption)
- The devsecops secrets-scanning, kubernetes-rbac-audit, and cloud kms-and-secrets-management skills

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.