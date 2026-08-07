---
name: secure-runners
domain: 08-devsecops-and-cicd-security
description: Use when securing CI/CD build runners — especially self-hosted ones that persist state and run untrusted code, closing a common foothold into the build environment.
difficulty: intermediate
tags: [devsecops, runners, cicd, isolation, self-hosted]
tools: [github-runners, gitlab-runners]
---

## Purpose

CI/CD runners are the machines that execute pipeline jobs — and they run whatever the pipeline tells them to, often with access to secrets and networks. Self-hosted runners especially can become a foothold: they persist state between jobs, may run untrusted PR code, and sit inside your network. This skill covers securing runners so a malicious or compromised job can't turn the runner into a persistent attack platform, closing a frequently-overlooked gap in pipeline security.

## When to use it

Securing any CI/CD setup, with particular urgency for self-hosted runners (cloud-hosted/ephemeral runners from the provider are more isolated by default). It complements pipeline hardening — the pipeline decides what runs; the runner is where it runs.

## Procedure

1. **Prefer ephemeral runners — the key mitigation.** A runner that's created fresh for each job and destroyed after leaves no persistent state for an attacker to establish a foothold in. Ephemeral (single-use) runners are the strongest defence against runner compromise; if a job is malicious, it gets a clean throwaway machine. Prefer them over long-lived persistent runners.
2. **Isolate runners from sensitive networks and resources.** A runner sits somewhere with network access; a self-hosted runner on your internal network is a pivot point if compromised. Place runners in an isolated network segment with least-privilege access, not on the corporate LAN with reach to sensitive systems.
3. **Don't run untrusted code on privileged runners — critical for self-hosted.** The classic self-hosted-runner attack: a malicious PR from a fork runs on your self-hosted runner (which has your secrets/network), executing the attacker's code in your environment. Never run untrusted PR code on runners with access to secrets or sensitive networks; require approval for fork-PR workflows, or use isolated ephemeral runners for untrusted builds.
4. **Least-privilege the runner's access.** Scope what secrets and resources the runner can reach to the minimum the jobs need. A runner with broad access turns any job compromise into broad compromise (the pipeline least-privilege principle, at the runner).
5. **Clean state between jobs.** On persistent runners, jobs can leave behind (or read) data from other jobs — secrets in memory/disk, cached credentials, malware. Clean the workspace and environment between jobs, or (better) use ephemeral runners that start clean every time.
6. **Keep runners patched and hardened** like any host (the Linux/Windows hardening domains) — they're machines that run code and are a target.
7. **Monitor runner activity.** Log what runs on runners and alert on anomalies (a job accessing unexpected resources, unusual network connections) — a compromised runner is a serious foothold worth detecting.

## Cheatsheet

```
runners = machines that RUN pipeline jobs (with secrets/network access)
  self-hosted especially = a foothold (persist state, run untrusted code, inside your net)

1. EPHEMERAL runners (the key mitigation): fresh per job, destroyed after
     -> no persistent state for a foothold ; malicious job gets a throwaway machine
2. ISOLATE from sensitive networks (self-hosted on corporate LAN = pivot point)
     -> isolated segment, least-privilege access
3. DON'T run untrusted code on privileged runners (critical, self-hosted)
     classic attack: malicious fork-PR runs on your runner w/ your secrets/network
     -> require approval for fork PRs / isolated ephemeral runners for untrusted builds
4. LEAST-PRIVILEGE runner access (broad access = job compromise -> broad compromise)
5. CLEAN state between jobs (persistent runners leak secrets/data across jobs) — or go ephemeral
6. PATCH + harden runners (they're hosts + targets)
7. MONITOR runner activity (compromised runner = serious foothold)
```

## Reading the setup

- **Persistent self-hosted runners running untrusted fork-PR code with secret/network access** = the classic, dangerous self-hosted-runner attack; an attacker's PR executes in your environment with your secrets. The highest-priority runner risk to close. Use ephemeral runners or require approval for untrusted builds.
- **Long-lived persistent runners** = state accumulates and can be a foothold; ephemeral single-use runners eliminate the persistent-foothold risk. The key mitigation.
- **A self-hosted runner on the corporate LAN** = a pivot point into your internal network if compromised; isolate runners in a segmented network with least-privilege reach.
- **Runners with broad secret/resource access** = any job compromise becomes broad compromise; scope runner access to the minimum.
- **Persistent runners not cleaned between jobs** = one job can read another's secrets or leave malware for the next; clean state, or use ephemeral runners that start fresh.
- **Ephemeral, isolated, least-privileged, monitored runners with untrusted builds separated** = runners that can't become a persistent attack platform.

## The fix / best practice

- **Use ephemeral (single-use) runners** wherever possible — the strongest defence, leaving no persistent foothold.
- **Isolate runners** in a segmented network with least-privilege access to secrets and resources.
- **Never run untrusted code (fork PRs) on privileged runners** — require approval or route untrusted builds to isolated ephemeral runners without secrets.
- **Clean state between jobs** on any persistent runner.
- **Patch and harden runners** as hosts, and **monitor** their activity for anomalies.
- Combine with pipeline hardening (least-privilege jobs, no secrets to fork PRs).

## Pitfalls

- **Running untrusted fork-PR code on privileged self-hosted runners.** The classic attack — the attacker's code runs with your secrets and network access. Never do it; isolate untrusted builds.
- **Long-lived persistent runners.** They accumulate state that becomes a foothold; ephemeral single-use runners are far safer. Prefer them.
- **Runners on sensitive networks.** A self-hosted runner on the corporate LAN is a pivot point if compromised; isolate it.
- **Broad runner access.** A runner that can reach many secrets/resources turns any job compromise into broad compromise; least-privilege it.
- **Not cleaning state.** Persistent runners leak secrets and data across jobs; clean between jobs or go ephemeral.
- **Treating runners as invisible infrastructure.** They run code and are a target; patch, harden, and monitor them.

## References

- GitHub Actions and GitLab CI self-hosted runner security documentation
- The pipeline-hardening skill and the Linux/Windows hardening domains
- OWASP CI/CD Security Top 10 (runner/execution risks)
- Poisoned pipeline execution research
