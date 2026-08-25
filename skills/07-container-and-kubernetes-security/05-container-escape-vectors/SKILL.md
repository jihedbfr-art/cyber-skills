---
format: "v2"
name: "container-escape-vectors"
title: "Container Escape Vectors"
title_fr: "Vecteurs d'évasion de conteneur"
description: "Use when assessing how a compromised container could break out to the host — the privileged, host-mount, and capability misconfigurations that turn a container compromise into node compromise."
description_fr: "À utiliser pour évaluer comment un conteneur compromis pourrait s'échapper vers l'hôte — les mauvaises configurations de privilèges, de montages hôte et de capacités qui transforment une compromission de conteneur en compromission de nœud."
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

Containers isolate workloads — until a misconfiguration lets one break out to the host. A container escape turns a compromised application into a compromised node, and in Kubernetes usually the whole cluster. This skill covers the configurations and vectors that enable escape, so you can recognise and eliminate them — understanding the attack (from the defender's side) to close the paths. It's the "why" behind the pod-security and Dockerfile hardening controls.

### When to use it

Assessing whether a container compromise could escalate to the host, hardening against escape, or in an authorised container-security assessment. Because escape is the difference between a contained incident and full node/cluster compromise, understanding these vectors is central to container security.

### The escape vectors (and the misconfigurations behind them)

Most escapes aren't kernel 0-days — they're misconfigurations that grant host access:

- **Privileged containers** (`privileged: true`) — the biggest one; near-total host access, trivially escapable (mount host disks, access host devices). A privileged container is effectively root on the node.
- **Sensitive host mounts** (`hostPath`) — mounting host directories, especially `/`, `/var/run/docker.sock` (the Docker socket = control the host's containers = host takeover), or `/proc`, gives paths to the host.
- **Excessive capabilities** — `CAP_SYS_ADMIN` and similar dangerous capabilities enable escape techniques; broad capabilities weaken the container boundary.
- **Host namespaces** (`hostPID`, `hostNetwork`, `hostIPC`) — sharing the host's namespaces breaks isolation and exposes host processes/network.
- **The Docker/container-runtime socket mounted into a container** — a classic and complete escape; access to the socket lets you launch a privileged container on the host.
- **Kernel vulnerabilities** — the harder path (a container-escape CVE in the kernel/runtime); real but far less common than misconfiguration. Keep hosts patched.

### Procedure

1. **Enumerate the container's privileges — assess escape potential.** From inside (or from the pod spec), check: is it privileged? what capabilities does it have? what's mounted from the host? what namespaces does it share? Tools like `amicontained` report the container's security context:
   ```
   amicontained          # reports capabilities, namespaces, seccomp, escape potential
   # or read the pod spec: securityContext, volumes (hostPath), hostPID/Network/IPC
   ```
2. **Check for the high-severity vectors first** — privileged, the Docker socket mounted, host root mounted, `CAP_SYS_ADMIN`. Any one of these is likely a full escape.
3. **Understand the escape (for defence)** — a privileged container or mounted socket lets an attacker gain host code execution; in an authorised test, demonstrate the *capability* rather than pivoting destructively across the node/cluster.
4. **Assess the blast radius.** An escape to the node is bad; in Kubernetes, a node has kubelet credentials and runs other pods, so node compromise usually escalates toward cluster compromise. Rate escapes by what the node/cluster exposes.
5. **Eliminate the misconfigurations — the fix is prevention.** These vectors are almost all closable by configuration: don't run privileged, don't mount the socket or host-sensitive paths, drop capabilities, don't share host namespaces. The pod-security-standards and Dockerfile-hardening skills enforce exactly this.
6. **Add runtime detection** for escape attempts (the runtime-threat-detection skill) as a backstop, and keep hosts/runtime patched for the kernel-vulnerability path.

### Cheatsheet

```
container isolates -> UNTIL a misconfig lets it break out. escape = container -> NODE -> cluster.
  most escapes = MISCONFIG, not kernel 0-days.

vectors (misconfig behind each)
  PRIVILEGED (privileged:true)   biggest — near-total host access, trivially escapable
  DOCKER/RUNTIME SOCKET mounted  classic complete escape (launch privileged container on host)
  HOST MOUNTS (hostPath: / /proc /var/run/docker.sock)  paths to the host
  CAP_SYS_ADMIN + dangerous caps enable escape techniques
  HOST NAMESPACES (hostPID/Network/IPC)  break isolation
  kernel/runtime CVE            harder, less common — keep hosts patched

assess: amicontained  (caps, namespaces, seccomp, escape potential)
  or read pod spec: securityContext.privileged, capabilities, volumes hostPath, host*

blast radius: node has kubelet creds + other pods -> node compromise -> cluster
FIX = PREVENTION (config): no privileged, no socket/host mounts, drop caps, no host namespaces
  (enforced by pod-security-standards + dockerfile-hardening)
+ runtime detection backstop + patch hosts
```

### Reading the assessment

- **A privileged container** = the highest-severity escape vector; it's effectively root on the node and trivially escapable. Any privileged workload is a critical finding — challenge and remove it.
- **The Docker/container-runtime socket mounted into a container** = a classic, complete escape; socket access lets an attacker launch a privileged container on the host. Never mount it into workloads.
- **Host-sensitive `hostPath` mounts** (`/`, `/proc`, the socket) = direct paths to the host; escape-enabling. Remove them.
- **`CAP_SYS_ADMIN` or broad capabilities** = weakens the container boundary and enables escape techniques; drop capabilities to the minimum.
- **Host namespaces shared** (`hostPID`/`hostNetwork`/`hostIPC`) = broken isolation exposing host processes and network. Avoid.
- **An escape demonstrated in a test** = rate by blast radius; in Kubernetes a node escape typically reaches the cluster via kubelet credentials and co-located pods. Escapes are rarely "just one node".
- **A container with no privileged flag, no host mounts/namespaces, minimal capabilities** = the hardened state where escape is genuinely hard.

### The fix

Prevention through configuration closes nearly all escape vectors:

- **Never run privileged containers** for workloads; it's the single biggest vector.
- **Never mount the container-runtime socket** or host-sensitive paths into containers.
- **Drop capabilities** to the minimum (drop all, add back only what's needed); avoid `CAP_SYS_ADMIN`.
- **Don't share host namespaces** (`hostPID`/`hostNetwork`/`hostIPC`).
- **Enforce these with Pod Security Standards / admission control** so escape-enabling pods are rejected at deployment (the pod-security and admission-control skills).
- **Patch hosts and the container runtime** for the kernel/runtime-CVE path, and add **runtime detection** as a backstop for escape attempts.

### Pitfalls

- **Running privileged "because it's easier".** It's the top escape vector and near-root on the node; the convenience is rarely worth cluster compromise risk. Find the specific capability instead.
- **Mounting the Docker socket into containers.** A complete, classic escape — socket access is host control. Never do it for workloads; use safer alternatives for socket-needing tools.
- **Broad capabilities / CAP_SYS_ADMIN.** They enable escape techniques; drop to the minimum.
- **Focusing on kernel exploits.** Real but uncommon; the overwhelming majority of escapes are misconfiguration. Fix the config first.
- **Underrating blast radius.** A node escape in Kubernetes usually reaches the cluster; don't treat it as "just one host".
- **Preventing without detecting.** Configuration prevention is primary, but add runtime detection for escape attempts as defence in depth.

### References

- MITRE ATT&CK for Containers (Escape to Host, T1611)
- amicontained and container-security assessment tooling
- The pod-security-standards, dockerfile-hardening, admission-control, and runtime-threat-detection skills
- NIST SP 800-190 (Application Container Security)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.