---
format: "v2"
name: "malicious-package-response"
title: "Malicious Package Response"
title_fr: "Réponse aux paquets malveillants"
description: "Use when a dependency you use turns out to be malicious or compromised — the response to a supply-chain incident where the threat is inside a package you already trusted and installed."
description_fr: "À utiliser lorsqu'une dépendance déjà utilisée s'avère malveillante ou compromise — la réponse à un incident de la chaîne d'approvisionnement où la menace se trouve à l'intérieur d'un paquet déjà installé et de confiance."
domain: "09-software-supply-chain-security"
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

Sometimes a package you already use goes bad — a maintainer account is compromised and pushes a malicious version, a dependency is hijacked, or a package is revealed as malicious after you adopted it. This is a supply-chain incident with a twist: the malicious code is *inside software you already trusted, installed, and possibly shipped*. This skill covers responding to a malicious or compromised package — a scenario that's become common (event-stream, ua-parser-js, and others) and needs a specific response.

### When to use it

When you learn a dependency you use is malicious or compromised — from a security advisory, a scanner alert, or news of a package hijack. It combines supply-chain knowledge with the incident-response discipline, applied to the specific case where the threat rode in through a trusted dependency.

### Procedure

1. **Determine your exposure fast.** Do you actually use the affected package and version? This is where an SBOM pays off — query it to instantly answer "are we affected, and where" across your projects and deployments (the sbom-generation skill). Without an SBOM, this is a frantic manual hunt.
2. **Assess what the malicious code did.** Malicious packages typically: steal secrets/environment variables/credentials, exfiltrate data, install backdoors, or run on install (install scripts) or at runtime. Understand the specific package's malicious behaviour (from the advisory/analysis) to know what to assume compromised — if it steals credentials, assume those credentials are compromised.
3. **Contain — remove the malicious version.** Pin to a known-good (pre-compromise) version or remove the package. Rebuild and redeploy without the malicious code. But removal alone isn't enough if it already executed.
4. **Assume-breach for what the code could access.** This is the key difference from a normal vulnerability: the malicious code *ran* in your build and/or runtime with your access. If it could read secrets (env vars, CI credentials, cloud keys), **rotate them** — assume anything the package could reach is compromised. This is often the most important step and the most overlooked.
5. **Investigate for actual compromise.** Did it exfiltrate? Did it establish persistence? Check logs (network connections to the package's exfil destination, unexpected activity during builds/runtime) for evidence of what actually happened, not just what the code was capable of. Escalate to full incident response if there's evidence of active compromise.
6. **Check where it shipped.** If the malicious package made it into artifacts you released or deployed, the exposure extends to those — and possibly to *your* downstream consumers. You may need to notify them (you became a supply-chain link).
7. **Prevent recurrence.** Add the controls that would have caught or contained it — dependency scanning, install-script review, pinning/lockfile integrity, and reducing the blast radius (least-privilege builds so a malicious package can't reach much). Feed lessons into a blameless postmortem (IR domain).

### Cheatsheet

```
malicious code INSIDE software you already trusted + installed + maybe shipped (event-stream, ua-parser-js)
  supply-chain incident with a twist: the threat rode in through a TRUSTED dependency

1. EXPOSURE fast: do you use the affected package+version? -> SBOM answers instantly (else frantic hunt)
2. what did it DO? steal secrets/env/creds | exfiltrate | backdoor | run on install/runtime
     -> if it steals creds, ASSUME those creds compromised
3. CONTAIN: pin to known-good (pre-compromise) version / remove ; rebuild+redeploy
     (removal alone insufficient if it already RAN)
4. ASSUME-BREACH for what it could access (KEY difference from a normal vuln):
     it RAN with your access -> ROTATE any secrets it could reach (env/CI/cloud keys)
     often the most important + most overlooked step
5. INVESTIGATE actual compromise: exfil? persistence? logs (connections to exfil dest, odd build/runtime activity)
     evidence of active compromise -> full incident response
6. WHERE SHIPPED: in released/deployed artifacts? -> exposure extends + notify YOUR downstream consumers
7. PREVENT: dep scanning + install-script review + pinning/lockfile + least-privilege builds (blast radius)
     -> blameless postmortem
```

### Reading the incident

- **Confirmed use of the malicious package version** = you're exposed; the SBOM query that tells you this instantly is why SBOMs matter — without one, determining exposure across projects is slow and error-prone under time pressure.
- **A package that steals credentials/secrets** = assume those credentials are compromised and rotate them; the malicious code ran with your access, so what it could reach must be treated as breached. This assume-breach step is the key difference from a normal vulnerability and the most commonly missed.
- **The malicious version removed but secrets not rotated** = incomplete response; removal stops future execution but doesn't undo what already ran. If it exfiltrated credentials, they're still out there. Rotate.
- **Evidence of exfiltration or persistence in logs** = actual compromise, not just capability; escalate to full incident response (containment, eradication, the IR domain).
- **The malicious package shipped in your released artifacts** = your exposure extends to those deployments and possibly your downstream consumers; you became a supply-chain link and may need to notify.
- **Exposure determined via SBOM, malicious version removed, reachable secrets rotated, compromise investigated, downstream notified, prevention added** = a complete malicious-package response.

### Pitfalls

- **Just removing the package.** Removal stops future execution but doesn't address that the code already *ran* with your access. If it could reach secrets, they may be compromised regardless of removal. Assume-breach and rotate.
- **Not rotating reachable secrets.** The most overlooked step — malicious code that ran in your build/runtime could read env vars, CI credentials, and cloud keys; assume anything it could reach is compromised and rotate it.
- **No SBOM.** Determining exposure across projects and deployments without one is a slow, error-prone manual hunt exactly when speed matters. Generate SBOMs in advance.
- **Treating it as a normal vulnerability.** A malicious package *executed* with your access; the response is assume-breach (rotate, investigate), not just "update to a fixed version".
- **Ignoring downstream.** If it shipped in your artifacts, your consumers are exposed; you may have a notification obligation.
- **Not adding prevention.** Without dependency scanning, install-script review, pinning, and least-privilege builds, the next malicious package lands the same way. Feed lessons back.

### References

- The sbom-generation skill (exposure determination) and the incident-response domain (assume-breach, eradication, postmortem)
- Notable cases: event-stream, ua-parser-js, node-ipc (malicious/compromised package incidents)
- The dependency-scanning, typosquat-detection, and pipeline least-privilege skills
- OpenSSF and CISA supply-chain incident guidance

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.