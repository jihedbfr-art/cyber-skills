---
format: "v2"
name: "network-policies"
title: "Network Policies"
title_fr: "Politiques réseau"
description: "Use when segmenting pod-to-pod traffic in Kubernetes — applying network policies to default-deny east-west traffic so a compromised pod can't reach everything in the cluster."
description_fr: "À utiliser pour segmenter le trafic pod à pod dans Kubernetes — appliquer des network policies en deny-by-default sur le trafic est-ouest pour qu'un pod compromis ne puisse pas atteindre tout le cluster."
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

By default, every pod in a Kubernetes cluster can talk to every other pod — a flat network where one compromised pod can reach the whole cluster. Network Policies are Kubernetes' segmentation mechanism: they restrict which pods can communicate, so a foothold in one workload can't freely reach the databases, other tenants, or sensitive services. This skill covers applying network policies to segment east-west traffic, the cluster equivalent of network segmentation.

### When to use it

Hardening a Kubernetes cluster, especially multi-tenant or multi-workload ones where a compromised pod shouldn't reach everything. It's a high-value control that's frequently missing because the default (allow-all) works, so nobody notices the lack of segmentation until an incident.

### Procedure

1. **Understand the default: allow-all.** Without network policies, all pod-to-pod traffic is permitted — a flat internal network. A compromised pod (via an app vuln) can then reach every other pod, service, and often the cluster's sensitive components. This is the gap network policies close.
2. **Confirm your CNI enforces Network Policies.** Network Policies are only enforced if your CNI plugin supports them (Calico, Cilium, etc.). Some CNIs don't enforce them — in which case a policy is silently ineffective. Verify enforcement is real, or the "segmentation" is theatre.
3. **Adopt default-deny per namespace — the key move.** Apply a default-deny policy so no traffic is allowed unless explicitly permitted, then add policies allowing the specific flows each workload needs. Default-deny plus explicit allows is real segmentation; without a default-deny, unlisted traffic is still allowed:
   ```
   # default-deny ingress for a namespace, then allow specific flows
   ```
4. **Define the allowed flows by workload need.** Allow the app tier to reach the database tier on the right port, allow ingress to the front-end, and deny the rest. Use pod/namespace selectors to express "these pods may talk to those pods" — least-connectivity, like firewall rules for pods.
5. **Segment tenants and sensitive workloads hardest.** In multi-tenant clusters, isolate namespaces from each other; keep databases and sensitive services reachable only from the specific workloads that need them.
6. **Control egress too, not just ingress.** Egress policies restrict what pods can connect *out* to — limiting a compromised pod's ability to reach C2 or exfiltrate. Often overlooked; ingress-only policies leave outbound open.
7. **Verify enforcement.** Test that a pod which *shouldn't* reach another actually can't — deploy a test and confirm the connection is denied. An unenforced or misconfigured policy is no segmentation.

### Cheatsheet

```
DEFAULT: every pod can talk to every pod (flat net) -> one compromised pod reaches all
  Network Policies = K8s segmentation (restrict pod-to-pod)

1. CNI must ENFORCE policies (Calico/Cilium/...) — some don't -> policy silently useless. VERIFY.
2. DEFAULT-DENY per namespace (the key move), then allow explicit flows
     without default-deny, unlisted traffic still ALLOWED (not real segmentation)
3. allow flows by NEED (app->db on port X, ingress->frontend, deny rest)
     pod/namespace SELECTORS = "these pods may talk to those"
4. segment TENANTS + sensitive workloads hardest (isolate namespaces ; db reachable only by app)
5. EGRESS too (not just ingress): limit outbound -> blunts C2/exfil (often overlooked)
6. VERIFY: test that a pod which SHOULDN'T reach another actually can't
```

### Reading the posture

- **No network policies (flat cluster network)** = a compromised pod can reach every other pod and service; the segmentation gap that's common because allow-all "just works". Applying default-deny with explicit allows is the fix and a high-value one.
- **A CNI that doesn't enforce Network Policies** = your policies are silently ineffective; the cluster is flat regardless of the YAML you wrote. Verify enforcement, or the segmentation is theatre.
- **Policies without a default-deny** = unlisted traffic is still allowed; you've allowed some flows but not restricted the rest. Real segmentation requires default-deny plus explicit allows.
- **Databases/sensitive services reachable from any pod** = a compromised front-end reaches the crown jewels; scope them to only the workloads that need them.
- **Ingress-only policies** = outbound is unrestricted, so a compromised pod can still reach C2 and exfiltrate. Add egress policies.
- **A test pod that shouldn't reach another but can** = the policy isn't enforced or is misconfigured; verify denials, since unenforced policy is no control.
- **Default-deny namespaces with least-connectivity allows and egress control** = real cluster segmentation; a foothold is contained.

### The fix / best practice

- **Default-deny per namespace, then explicit allows** for the flows each workload needs — this is what makes it real segmentation.
- **Verify your CNI enforces Network Policies** (Calico, Cilium, etc.), or switch to one that does.
- **Least-connectivity between workloads** using pod/namespace selectors; isolate tenants and keep sensitive services reachable only from their consumers.
- **Control egress**, not just ingress, to blunt C2 and exfiltration from a compromised pod.
- **Verify enforcement** by testing that disallowed connections are actually blocked.
- Combine with pod-security, RBAC, and runtime detection for defence in depth.

### Pitfalls

- **No network policies at all.** The default flat network lets one compromised pod reach everything; it's the most common cluster-segmentation gap. Apply default-deny plus allows.
- **A non-enforcing CNI.** Policies only work if the CNI enforces them; otherwise they're silently useless and the cluster stays flat. Verify enforcement.
- **Allows without default-deny.** Allowing some flows without denying the rest leaves unlisted traffic permitted — not real segmentation. Default-deny is the foundation.
- **Ingress-only.** Leaving egress unrestricted lets a compromised pod reach C2 and exfiltrate; control outbound too.
- **Not verifying.** An unenforced or misconfigured policy protects nothing; test that disallowed connections are actually blocked.
- **Sensitive services open to all pods.** Databases reachable from any workload turn any pod compromise into data access; scope them tightly.

### References

- Kubernetes Network Policies documentation; Calico and Cilium network policy
- The network-segmentation skill (same principle at the network layer) and CIS Kubernetes Benchmark
- The container-escape-vectors and kubernetes-rbac-audit skills (defence in depth)
- MITRE ATT&CK for Containers (lateral movement)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.