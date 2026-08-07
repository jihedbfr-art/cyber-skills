---
name: pod-security-standards
domain: 07-container-and-kubernetes-security
description: Use when enforcing secure pod configurations in Kubernetes — the Pod Security Standards that stop privileged, host-accessing pods that lead to container escape and node compromise.
difficulty: intermediate
tags: [kubernetes, pod-security, hardening, container-escape, admission]
tools: [kubectl]
---

## Purpose

A pod can be configured to do almost anything to its node — run privileged, mount the host filesystem, use the host network, or run as root — and each of those is a path from a compromised container to a compromised node and cluster. Kubernetes Pod Security Standards define levels of restriction that block these dangerous configurations. This skill covers enforcing them so pods can't be deployed with the settings that enable container escape.

## When to use it

Hardening a Kubernetes cluster, and as a baseline control alongside RBAC. Restricting pod configuration is one of the most important cluster hardening steps because it directly closes the container-escape vectors (that skill covers the vectors; this closes them at the configuration gate).

## The Pod Security Standards

Kubernetes defines three levels, enforced via Pod Security Admission:

- **Privileged** — unrestricted; allows everything including privileged containers and host access. This is the dangerous level to avoid for workloads.
- **Baseline** — blocks the most dangerous settings (privileged containers, host namespaces, most host mounts) while staying broadly compatible. A reasonable minimum.
- **Restricted** — heavily restricted, following hardening best practice: non-root, no privilege escalation, dropped capabilities, restricted volumes, read-only where possible. The target for security-sensitive workloads.

## Procedure

1. **Understand the dangerous pod settings you're restricting** — these are the container-escape enablers:
   - `privileged: true` — near-total host access; a privileged container is effectively root on the node.
   - `hostPID`/`hostNetwork`/`hostIPC` — sharing host namespaces breaks isolation.
   - `hostPath` volumes — mounting host directories (especially sensitive ones) enables escape.
   - running as root / `allowPrivilegeEscalation: true` / broad capabilities.
2. **Enforce a Pod Security Standard level via Pod Security Admission.** Label namespaces to enforce baseline or restricted, so pods violating the level are rejected at admission:
   ```
   kubectl label namespace <ns> pod-security.kubernetes.io/enforce=restricted
   ```
   Use `warn`/`audit` modes first to see what would break before enforcing.
3. **Target `restricted` for workloads that can meet it**, `baseline` as the minimum floor. Privileged should be reserved for the rare system component that genuinely needs it, in its own namespace.
4. **Set the pod securityContext explicitly** — `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, drop all capabilities (add back only what's needed), `readOnlyRootFilesystem: true` where possible. This is how a pod meets the restricted standard.
5. **Roll out carefully.** Enforcing restricted can break workloads that assume root or need a capability; use warn/audit mode to find them, fix the workloads, then enforce. This is the same canary discipline as any hardening.
6. **Combine with admission control for custom policy** where Pod Security Standards aren't granular enough (the admission-control skill with OPA/Kyverno).
7. **Verify** that dangerous pods are actually rejected — deploy a test privileged pod and confirm it's blocked; an unenforced standard is no control.

## Cheatsheet

```
a pod can do almost anything to its NODE (privileged, host mount, host net, root)
  -> each = a path from compromised container to compromised NODE/cluster
  Pod Security Standards block these at ADMISSION

levels (via Pod Security Admission)
  Privileged   unrestricted — AVOID for workloads
  Baseline     blocks the worst (privileged, host namespaces, most host mounts) — min floor
  Restricted   hardening best practice (non-root, no-priv-esc, drop caps, restricted vols) — target

dangerous settings you're blocking (container-escape enablers)
  privileged: true | hostPID/hostNetwork/hostIPC | hostPath volumes
  | root / allowPrivilegeEscalation: true / broad capabilities

enforce: kubectl label ns X pod-security.kubernetes.io/enforce=restricted
  use warn/audit FIRST -> find what breaks -> fix workloads -> enforce
securityContext: runAsNonRoot, allowPrivilegeEscalation:false, drop ALL caps,
  readOnlyRootFilesystem
VERIFY: test privileged pod is REJECTED (unenforced standard = no control)
```

## Reading the posture

- **Namespaces at `privileged` (or no Pod Security enforcement)** = pods can be deployed with any dangerous setting, so container escape to the node is wide open. Enforcing baseline/restricted is the fix and a top priority.
- **A workload running `privileged: true`** = effectively root on the node; a compromise of that container is a compromise of the node and likely the cluster. Rarely justified — challenge every one.
- **`hostPath` mounts or host namespaces (`hostPID`/`hostNetwork`)** = isolation-breaking configurations that enable escape; the restricted standard blocks them.
- **Pods running as root without securityContext hardening** = escape/compromise is root; `runAsNonRoot` and dropped capabilities are what meet the restricted standard.
- **An enforced standard that a test privileged pod slips past** = the enforcement isn't actually applied (wrong label, wrong mode); verify rejection, since an unenforced standard protects nothing.
- **Namespaces enforcing restricted with hardened securityContexts** = the strong state; dangerous pod configs are blocked at admission.

## The fix / best practice

- **Enforce a Pod Security Standard** — restricted for workloads that can meet it, baseline as the floor; reserve privileged for the rare genuine system need in an isolated namespace.
- **Set hardened securityContexts** — non-root, no privilege escalation, dropped capabilities, read-only root filesystem — so pods meet the restricted level.
- **Roll out via warn/audit first**, fix the workloads that break, then enforce.
- **Verify enforcement** by confirming a dangerous test pod is rejected.
- **Use admission control (OPA/Kyverno)** for policy beyond what Pod Security Standards express.
- Combine with the RBAC audit and network policies for defence in depth.

## Pitfalls

- **No enforcement.** Without a Pod Security Standard enforced, pods can use any dangerous setting; the escape vectors are wide open. Enforce baseline/restricted.
- **Allowing privileged workloads.** `privileged: true` is near-root on the node and rarely justified; challenge every use and isolate the exceptions.
- **Enforcing restricted without a canary.** It can break workloads assuming root or needing capabilities; use warn/audit mode, fix, then enforce.
- **Not verifying.** A mislabeled namespace or wrong mode means the standard isn't enforced; confirm a dangerous test pod is actually rejected.
- **Relying on Pod Security Standards alone.** They're coarse; use admission control for granular policy and combine with RBAC and network policy.

## References

- Kubernetes Pod Security Standards and Pod Security Admission documentation
- CIS Kubernetes Benchmark (pod security controls)
- The container-escape-vectors, admission-control, and kubernetes-rbac-audit skills
- MITRE ATT&CK for Containers (escape to host)
