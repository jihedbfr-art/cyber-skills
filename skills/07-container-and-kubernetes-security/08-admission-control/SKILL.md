---
format: "v2"
name: "admission-control"
title: "Admission Control"
title_fr: "Contrôle d'admission"
description: "Use when enforcing security policy at deploy time in Kubernetes — admission controllers (OPA/Kyverno) that reject non-compliant resources before they run, gating the cluster."
description_fr: "À utiliser pour imposer une politique de sécurité au moment du déploiement dans Kubernetes — des admission controllers (OPA/Kyverno) qui rejettent les ressources non conformes avant qu'elles ne s'exécutent, verrouillant l'accès au cluster."
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

The most reliable way to keep dangerous configurations out of a cluster is to reject them at the door — before they're ever created. Admission controllers intercept every resource request to the Kubernetes API and can validate it against policy, blocking non-compliant resources (a privileged pod, an image from an untrusted registry, a missing security context) at deploy time. This skill covers using admission control (OPA/Gatekeeper, Kyverno) to enforce security policy as code, the gate that makes the other container controls non-optional.

### When to use it

Hardening a cluster to *enforce* rather than just recommend security policy. It's what turns pod-security, image, and RBAC best practices from guidelines into hard requirements — non-compliant resources simply can't be deployed. Essential for any cluster where you can't manually review every deployment.

### Procedure

1. **Understand admission control's place.** When a resource is created/updated, admission controllers run before it's persisted — **validating** (accept/reject) and optionally **mutating** (modifying, e.g. adding a default security context). This deploy-time gate is where you enforce policy consistently, rather than hoping every manifest is written correctly.
2. **Deploy a policy engine.** OPA/Gatekeeper (policy in Rego) or Kyverno (policy in YAML, Kubernetes-native and often simpler) are the standard tools. They let you write policies as code, versioned and reviewed. Kyverno is usually easier to adopt for Kubernetes-native teams.
3. **Enforce the key security policies:**
   - **Reject dangerous pod configs** — privileged, host mounts/namespaces, running as root (complementing Pod Security Standards with more granular rules).
   - **Require trusted images** — only from approved registries, and (with signing) only signed images (the supply-chain skill).
   - **Require security context** — non-root, dropped capabilities, resource limits set.
   - **Enforce labels/ownership, block risky defaults**, and require network policies exist.
4. **Roll out in audit mode first.** Run policies in audit/warn mode to see what would be rejected before enforcing — enforcing untested policies breaks deployments. Fix the non-compliant workloads, then switch to enforce.
5. **Use mutation carefully** — admission controllers can auto-add secure defaults (a default securityContext, dropping capabilities), which helps compliance, but understand mutations change what users deployed; keep them predictable and documented.
6. **Version policies as code and test them.** Admission policies are code — store in git, review changes, and test that they reject what they should and allow legitimate resources. A policy that blocks legitimate deploys causes outages; one that's too loose lets bad configs through.
7. **Monitor and maintain.** Track policy violations (what's being rejected) as a signal, and maintain policies as the cluster and threats evolve.

### Cheatsheet

```
best way to keep dangerous configs out = reject them at the DOOR (deploy time, before running)
  admission controllers intercept API requests -> VALIDATE (accept/reject) + MUTATE (modify)
  makes the other container controls NON-OPTIONAL

engines: OPA/Gatekeeper (Rego) | Kyverno (YAML, K8s-native, often simpler)

enforce
  reject dangerous pods: privileged / host mounts+namespaces / root (granular, complements PSS)
  require TRUSTED images: approved registries only ; SIGNED only (supply-chain)
  require securityContext: non-root, drop caps, resource limits
  enforce labels/ownership, require network policies, block risky defaults

roll out: AUDIT/warn mode first -> find non-compliant -> fix -> ENFORCE
  (enforcing untested policies breaks deploys)
mutation: auto-add secure defaults — helpful but changes what users deployed (predictable+documented)
policies as CODE: git + review + TEST (too strict = outage ; too loose = bad configs through)
monitor violations ; maintain
```

### Reading the setup

- **No admission control** = security policy is advisory only; a privileged pod, an untrusted image, or a root container can be deployed freely, relying on everyone writing manifests correctly. Admission control is what makes policy enforceable — a high-value addition.
- **Policies enforcing "reject privileged / require non-root / trusted registries only"** = the dangerous configurations can't be deployed at all; this is the gate that makes pod-security and image controls hard requirements.
- **Policies enforced without an audit-mode rollout** = they may block legitimate deployments and cause outages; audit first, fix workloads, then enforce. The classic mistake.
- **Image policies requiring signed images from approved registries** = closes the supply-chain gap at deploy time; only vetted images run.
- **A policy that's too loose** (allows what it should block) = false comfort; test that policies actually reject non-compliant resources.
- **Versioned, tested, monitored admission policies enforcing the key controls** = the deploy-time gate working; the cluster enforces its security posture automatically.

### The fix / best practice

- **Deploy an admission policy engine** (Kyverno or OPA/Gatekeeper) and enforce the key security policies as code.
- **Reject dangerous pod configs, require trusted/signed images, require hardened security contexts** — making the other container controls non-optional.
- **Roll out in audit mode first**, fix non-compliant workloads, then enforce.
- **Version policies in git, review, and test** them (reject the bad, allow the legitimate).
- **Use mutation for secure defaults** carefully and predictably.
- **Monitor violations** and maintain policies as the environment evolves.

### Pitfalls

- **No admission control.** Without it, security policy is just advice; dangerous configs deploy freely. It's the enforcement that makes the other controls real.
- **Enforcing untested policies.** Policies that block legitimate deployments cause outages; roll out in audit mode, fix, then enforce.
- **Too-loose policies.** A policy that doesn't actually reject the bad config gives false comfort; test enforcement.
- **Unpredictable mutations.** Auto-modifying resources helps compliance but changes what users deployed; keep mutations predictable and documented.
- **Set-and-forget.** Clusters and threats evolve; policies need maintenance, and violation trends are a useful signal.

### References

- Kyverno and OPA/Gatekeeper documentation; Kubernetes admission controllers reference
- The pod-security-standards, supply-chain-for-images, and kubernetes-rbac-audit skills
- The devsecops policy-as-code skill (same discipline) and CIS Kubernetes Benchmark
- MITRE ATT&CK for Containers

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.