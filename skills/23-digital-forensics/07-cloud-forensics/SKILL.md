---
name: cloud-forensics
domain: 23-digital-forensics
description: Use when investigating in the cloud where there's no disk to image — reconstructing events from audit logs, snapshots, and provider APIs instead of traditional acquisition.
difficulty: advanced
tags: [forensics, cloud, audit-logs, snapshots, aws]
tools: [aws-cli]
---

## Purpose

Traditional forensics assumes you can seize and image a machine. In the cloud, you often can't — the "host" is ephemeral, you don't control the hardware, and an instance may be gone before you look. But the cloud gives you something powerful in exchange: a detailed API audit log of nearly everything that happened, plus the ability to snapshot storage on demand. This skill covers investigating cloud environments on their terms — logs and snapshots over disk imaging. It's the forensics counterpart to the cloud-incident-response skill.

## When to use it

Investigating incidents involving cloud resources — a compromised instance, an abused IAM identity, a data-exposure event. The approach differs enough from traditional forensics that applying host-forensics habits directly will miss the evidence, which lives in the control plane.

## Procedure

1. **The audit log is the primary evidence — preserve it first.** CloudTrail / Activity Log / Cloud Audit Logs record API calls: who did what, when, from where. This is the cloud equivalent of the crime scene, and reconstruction depends on it (which is why the cloudtrail-and-audit-logging skill's setup is a prerequisite — you can only investigate what was being logged). Export and protect the relevant logs before anything changes them:
   ```
   aws cloudtrail lookup-events --lookup-attributes AttributeKey=Username,AttributeValue=<principal>
   ```
2. **Snapshot volatile resources before they vanish.** An instance may terminate (or be terminated) and take its disk with it. Snapshot the EBS volumes / disks of affected instances immediately to preserve them for analysis — the cloud way of "imaging". Capture memory too where the platform allows.
3. **Analyse the snapshot like a disk image.** Attach the snapshot to a forensic workstation (a separate, isolated analysis instance) and run traditional disk forensics on it (the disk/artefact skills apply once you have the volume).
4. **Reconstruct from the control plane.** Much of the cloud story is in the logs, not on any disk: IAM changes, resource creation/deletion, data access (S3 object-level events if enabled), and network changes. Trace the compromised identity's actions across the audit log to build the timeline.
5. **Use provider forensic features.** Cloud platforms increasingly offer forensic tooling — automated snapshotting, isolation, and detection services (GuardDuty/Defender/Security Command Center findings) — use them, and check what managed logging/telemetry already captured.
6. **Mind the shared-responsibility boundary.** You can investigate your workloads and your account's logs, but not the provider's underlying infrastructure. Some evidence (hypervisor-level) is only obtainable via the provider, sometimes through legal process.
7. **Preserve chain of custody for cloud evidence** — logs exported, snapshots taken, hashes recorded, and API actions documented, exactly as for physical evidence.

## Cheatsheet

```
cloud forensics != disk imaging. two pillars:

1. AUDIT LOG (primary evidence — preserve FIRST)
   CloudTrail / Activity Log / Cloud Audit Logs = who/what/when/where (API calls)
   reconstruct the compromised identity's actions -> timeline
   (prereq: logging was configured beforehand — cloudtrail-and-audit-logging skill)
   attacker log-tampering (StopLogging/DeleteTrail) = a finding + a blind-spot marker

2. SNAPSHOTS (volatile — capture before the resource vanishes)
   snapshot affected EBS/disks immediately (instances can terminate + take disk)
   attach to an isolated forensic instance -> traditional disk forensics applies
   memory capture where the platform allows

provider tooling: automated snapshot/isolation, GuardDuty/Defender/SCC findings
shared responsibility: your workloads+logs yes ; provider infra = via provider/legal
chain of custody: export logs, snapshot, HASH, document API actions
```

## Reading the investigation

- **A complete, intact audit log** = you can reconstruct the incident precisely from the control plane — the cloud's big advantage over on-prem. Preserve it before the attacker (or a cleanup) alters it.
- **`StopLogging`/`DeleteTrail` in the history** = the attacker tried to blind the investigation; treat activity after that point as unlogged, and widen the scope. The tampering itself is evidence.
- **An instance already terminated before you snapshotted** = disk evidence likely lost; this is why snapshotting affected resources *immediately* is critical — cloud resources are volatile and don't wait.
- **IAM changes, resource creation in odd regions, unusual data access in the logs** = the attacker's actions reconstructed from the control plane, often the bulk of the story since there may be no host to examine.
- **Logging that wasn't configured beforehand** = a serious evidence gap; you can't retroactively log past events. The investigation is constrained by what was captured, underscoring the prerequisite.
- **Snapshot analysed + control-plane timeline built + custody preserved** = a sound cloud investigation.

## Pitfalls

- **Applying host-forensics habits.** Waiting to "image the machine" while the ephemeral instance terminates loses the disk. Snapshot immediately and lean on the audit log — the evidence model is different.
- **Assuming logging was on.** If CloudTrail/audit logging wasn't configured before the incident, control-plane reconstruction may be impossible. This is why the logging setup is a prerequisite, not an afterthought.
- **Not preserving logs before they change.** An attacker with access may tamper with or stop logging; export and protect the evidence early.
- **Terminating instead of snapshotting.** Deleting a compromised instance destroys its disk evidence; snapshot and isolate, don't delete.
- **Overstepping the shared-responsibility boundary.** You can't image the provider's hardware; some evidence requires the provider or legal process — know the limit.

## References

- AWS/Azure/GCP incident response and forensics guidance; cloud provider audit-log docs
- The cloudtrail-and-audit-logging (prerequisite), cloud-incident-response, and disk-imaging-and-hashing skills
- SANS cloud forensics resources; MITRE ATT&CK for Cloud
- Shared responsibility model documentation
