---
format: "v2"
name: "cloud-incident-response"
title: "Cloud Incident Response"
title_fr: "Réponse à incident dans le cloud"
description: "Use when responding to an incident in a cloud environment where there's no server to unplug — the API-driven, identity-centric response that differs from on-prem IR."
description_fr: "À utiliser lors d'un incident survenant dans un environnement cloud, où il n'y a pas de serveur à débrancher — une réponse pilotée par API et centrée sur l'identité, différente de l'IR sur site."
domain: "22-incident-response"
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

Cloud incident response breaks a lot of on-prem instincts. There's often no physical host to seize, the attacker moved through API calls rather than a network, and the "kill switch" is an IAM change, not a network cable. But the cloud also gives you something on-prem rarely does: a near-complete API audit log of everything that happened. This skill covers responding to a cloud incident on its own terms.

### When to use it

Any incident where the affected assets are cloud resources — a compromised access key, a public bucket that got hit, an over-permissioned role that was abused, crypto-mining on your compute. It builds on the general IR lifecycle skills but changes how each phase works.

### What's different from on-prem

- **The log is the crime scene.** CloudTrail / Activity Log / Cloud Audit Logs record almost every API call. Reconstruction is possible in a way it rarely is on-prem — *if* logging was configured beforehand (see the cloudtrail skill).
- **Identity is the perimeter.** The attacker usually acted as a credential or role, not from a network location. Containment centres on keys, roles, and sessions.
- **Containment is an API call.** You isolate by revoking credentials, attaching a deny policy, or snapshotting and quarantining an instance — not by pulling cables.
- **Resources are ephemeral.** An instance may be gone before you look. Snapshot before it disappears; evidence you don't capture is often unrecoverable.

### Procedure

1. **Preserve the logs first.** Confirm the audit trail is intact and protect it — an attacker with enough access may try to stop logging or delete the trail. Export/lock the relevant CloudTrail data before anything else; it's your primary evidence.
2. **Scope via the audit log.** Reconstruct what the compromised identity did: which API calls, in which regions, against which resources, over what window. `lookup-events` filtered by the principal is the starting point:
   ```
   aws cloudtrail lookup-events --lookup-attributes AttributeKey=Username,AttributeValue=<principal>
   ```
   Look for the tell-tale attacker actions: `CreateUser`, `CreateAccessKey`, `AttachUserPolicy`, `CreateRole`, resource creation in unusual regions.
3. **Contain the identity.** Revoke the compromised access key, and revoke *active sessions* (an IAM policy that denies actions before a cutoff timestamp) — deactivating a key doesn't kill sessions already assumed from it. Detach abused permissions.
4. **Preserve resource evidence before it vanishes.** Snapshot affected instances/volumes, and isolate rather than terminate — a terminated instance takes its evidence with it. Move it to a quarantine security group with no egress instead of deleting it.
5. **Hunt for cloud-native persistence** — the attacker's footholds live in the control plane, not just on hosts: rogue IAM users/roles, added access keys, modified trust policies, new Lambda functions, altered `CloudTrail`/logging config, resource-based policy changes, and cross-account roles.
6. **Eradicate and recover** per the eradication skill, cloud-flavoured: remove the rogue identities and persistence, rotate all potentially-exposed credentials, close the entry vector (the leaked key, the SSRF, the public resource), and rebuild from known-good IaC.
7. **Check the blast radius across accounts.** In a multi-account org, a compromise can pivot via assumed roles — scope beyond the single account.

### Cheatsheet

```bash
aws cloudtrail lookup-events --lookup-attributes AttributeKey=Username,AttributeValue=PRINCIPAL
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=CreateAccessKey

CreateUser / CreateAccessKey / CreateLoginProfile   (new footholds)
AttachUserPolicy / PutUserPolicy / CreateRole        (privilege)
UpdateAssumeRolePolicy                               (backdoor trust)
StopLogging / DeleteTrail                            (evidence tampering)
RunInstances in unusual regions                      (crypto-mining)

aws iam update-access-key --access-key-id AKIA... --status Inactive

aws ec2 create-snapshot --volume-id vol-...          # snapshot first
```

### Reading the situation

- **Logging was on and intact** = you can reconstruct the incident precisely; the audit trail is your biggest advantage. Preserve it before the attacker can.
- **`StopLogging`/`DeleteTrail` in the history** = the attacker tried to blind you; assume activity after that point is unlogged and widen your investigation.
- **New IAM users, keys, or modified trust policies** = cloud-native persistence — a deactivated key means nothing if a rogue role remains. Hunt the control plane, not just hosts.
- **A deactivated key but continued attacker activity** = sessions were already assumed; you must deny by timestamp, not just disable the key.
- **Resources in unfamiliar regions** = a classic tell (crypto-mining, staging), and easily missed if you only look where your workloads normally run.

### Pitfalls

- **Terminating instead of snapshotting.** A terminated instance is gone with its evidence. Snapshot, then isolate.
- **Deactivating a key but not revoking sessions.** Credentials already assumed keep working; kill active sessions with a deny-before-timestamp policy.
- **Only looking at hosts.** Cloud persistence is in IAM and the control plane — rogue roles, keys, trust policies, Lambda. Missing them means the attacker stays.
- **Assuming logging was configured.** If the audit trail wasn't set up beforehand, reconstruction may be impossible — which is why the cloudtrail skill is a prerequisite, not an afterthought.
- **Single-account tunnel vision.** Assumed-role pivots cross account boundaries; scope the whole org.

### References

- NIST SP 800-61r2 (incident handling) adapted to cloud
- AWS Incident Response guide / Cloud Security Incident Response whitepaper
- Cloud provider audit-log documentation (CloudTrail, Azure Activity Log, GCP Audit Logs)
- MITRE ATT&CK for Cloud

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.