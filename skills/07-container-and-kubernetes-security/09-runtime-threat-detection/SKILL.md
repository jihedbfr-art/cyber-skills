---
name: runtime-threat-detection
domain: 07-container-and-kubernetes-security
description: Use when detecting malicious behaviour in running containers — the runtime monitoring that catches escapes, crypto-mining, and anomalous activity that build-time controls can't.
difficulty: intermediate
tags: [containers, runtime, detection, falco, behavioural]
tools: [falco, tetragon]
---

## Purpose

Scanning images and enforcing policy at deploy time keeps a lot of bad things out — but it can't catch what happens *after* a container starts: an exploited application, a container escape attempt, crypto-mining, or an attacker's post-compromise activity. Runtime threat detection watches running containers for malicious behaviour, providing the detection layer that build-time and deploy-time controls can't. This skill covers runtime monitoring for containers and Kubernetes, closing the gap after the container is live.

## When to use it

As the runtime layer of container security, complementing image scanning (build), admission control (deploy), and hardening (config). It's what detects the compromise that gets through those — because prevention is never perfect, you need detection for running workloads.

## Procedure

1. **Understand what runtime detection adds.** Build/deploy controls are preventive and static; runtime detection is behavioural and live. It catches: an application exploited at runtime, a container escape attempt, unexpected process execution (a shell spawned in a container that shouldn't have one), crypto-mining, connections to C2, and file/privilege changes — activity that only exists once the container runs.
2. **Deploy a runtime security tool.** Falco (the CNCF standard, using kernel-level syscall monitoring) or eBPF-based tools (Tetragon, Cilium) observe container behaviour at the kernel level and alert on suspicious activity. These see what containers actually do, not just what they were configured to do.
3. **Detect the high-value behaviours:**
   - **Shell/unexpected process in a container** — a shell spawned in a container that should only run its app is a strong compromise signal (containers are usually single-purpose).
   - **Container escape attempts** — the escape vectors in action (privileged operations, sensitive mounts accessed, host namespace access).
   - **Crypto-mining** — high CPU plus mining-pool connections; a very common container compromise outcome.
   - **Unexpected network connections** — a container connecting to C2 or scanning internally.
   - **File and privilege changes** — writes to sensitive paths, privilege escalation, package installs at runtime.
4. **Baseline expected behaviour.** Containers are typically single-purpose and predictable, which makes anomaly detection more tractable than on general-purpose hosts — a container's normal behaviour is narrow, so deviation stands out. Baseline what each workload normally does.
5. **Tune to reduce noise.** Runtime tools can be noisy (legitimate but unusual activity); tune rules to your workloads, using the same false-positive discipline as detection engineering. Untuned runtime alerts get ignored.
6. **Wire alerts into the SOC and response.** Runtime detections feed the SOC (triage) and IR; a confirmed container compromise needs response (isolate the pod, investigate, eradicate). Detection without response is incomplete.
7. **Combine with the preventive controls.** Runtime detection is the backstop, not a replacement — a compromise it catches should also prompt fixing the preventive gap (the image vuln, the missing policy) that let it happen.

## Cheatsheet

```
build scan + deploy admission = PREVENTIVE/static. can't catch what happens AFTER start.
  runtime detection = BEHAVIOURAL/live — the detection layer for RUNNING containers

tools: Falco (CNCF, syscall/kernel-level) | Tetragon/Cilium (eBPF)
  -> see what containers actually DO, not just their config

high-value runtime behaviours
  SHELL / unexpected process in a container   strong compromise signal (containers = single-purpose)
  container ESCAPE attempts                   privileged ops, sensitive mounts, host namespace
  CRYPTO-MINING                               high CPU + mining-pool connections (common outcome)
  unexpected NETWORK (C2, internal scanning)
  FILE/PRIVILEGE changes                      sensitive writes, priv-esc, runtime package installs

BASELINE: containers are single-purpose + predictable -> anomaly detection tractable (deviation stands out)
TUNE noise (detection-engineering discipline) ; wire to SOC + IR (detection needs response)
backstop, not replacement -> also fix the preventive gap that let it in
```

## Reading the detections

- **A shell spawned inside a container that only runs an app** = one of the strongest container-compromise signals; containers are single-purpose, so an interactive shell is almost always an attacker or an exploited process. High-value detection.
- **Container escape-attempt behaviour** (privileged operations, sensitive mounts accessed, host namespace access at runtime) = an active attempt to break out to the node; escalate immediately and fix the config that allowed it.
- **Crypto-mining signals** (sustained high CPU + mining-pool connections) = a very common container compromise outcome; often the first visible sign that a container was popped.
- **Unexpected outbound connections** = C2 or internal scanning from a compromised container; the network behaviour betrays the compromise even if the process looks normal.
- **Runtime alerts that are noisy** = need tuning to the workload; untuned runtime detection gets ignored like any noisy detection. Apply false-positive discipline.
- **A runtime detection with no response path** = incomplete; a caught compromise needs isolation, investigation, and eradication. Wire it to IR.
- **Baselined, tuned runtime detection feeding the SOC, backing up preventive controls** = the complete picture — prevention plus detection for what gets through.

## Pitfalls

- **Relying only on build/deploy controls.** They're preventive and can't catch runtime compromise (exploited apps, escapes, mining). Prevention isn't perfect; you need runtime detection for what gets through.
- **No runtime monitoring.** A compromised running container is invisible without it; escapes and mining proceed unseen. Deploy Falco or an eBPF tool.
- **Untuned runtime alerts.** They're noisy and get ignored; tune to your workloads with false-positive discipline.
- **Detection without response.** A runtime alert that doesn't trigger investigation and containment is wasted; wire it to the SOC and IR.
- **Treating detection as a substitute for prevention.** It's a backstop — also fix the preventive gap (image vuln, missing policy, escape vector) that allowed the compromise.
- **Ignoring the single-purpose advantage.** Containers' predictability makes anomaly detection more effective than on general hosts; baseline and exploit that.

## References

- Falco (falco.org, CNCF) and Tetragon/Cilium eBPF documentation
- The container-image-scanning, admission-control, and container-escape-vectors skills
- The detection-engineering and SOC domains (tuning, triage, response)
- MITRE ATT&CK for Containers
