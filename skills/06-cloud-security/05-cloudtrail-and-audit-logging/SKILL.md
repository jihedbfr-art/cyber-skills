---
format: "v2"
name: "cloudtrail-and-audit-logging"
title: "Cloudtrail And Audit Logging"
title_fr: "CloudTrail et journalisation d'audit"
description: "Use when setting up or reviewing cloud audit logging — making sure API activity is recorded, protected from tampering, and actually usable during an investigation."
description_fr: "À utiliser pour mettre en place ou revoir la journalisation d'audit cloud — s'assurer que l'activité des API est bien enregistrée, protégée contre toute altération, et réellement exploitable lors d'une investigation."
domain: "06-cloud-security"
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

In the cloud, almost everything is an API call — and the audit log of those calls is how you detect misuse and investigate an incident. If it isn't turned on, isn't complete, or can be deleted by the attacker, you're blind exactly when it matters. This skill covers getting cloud audit logging right, using AWS CloudTrail as the concrete example (Azure Activity Log and GCP Cloud Audit Logs follow the same principles).

### When to use it

Standing up a cloud account, hardening an existing one, or preparing for incident response and compliance. Do it before an incident — reconstructing "who did what" is only possible if the logging was already correct when it happened.

### Procedure

1. **Confirm logging is on and org-wide.** A trail should capture every region and every account, so activity can't hide in an unmonitored corner:
   ```
   aws cloudtrail describe-trails
   aws cloudtrail get-trail-status --name <trail>
   ```
   Check `IsMultiRegionTrail` is true and the trail is logging.
2. **Confirm coverage.** Management events are the baseline; enable **data events** for sensitive resources (S3 object access, Lambda invokes) where you need object-level visibility — they're off by default and are often exactly what an investigation needs.
3. **Protect the logs from tampering.** The attacker's first move is often to delete evidence. Ensure:
   - logs go to a **dedicated, locked-down bucket** in a separate/logging account the workload roles can't touch;
   - **log file validation** is enabled (integrity hashes so tampering is detectable);
   - bucket versioning/Object Lock and MFA-delete guard against deletion.
4. **Make it usable.** Ship logs somewhere you can query (CloudWatch Logs, a SIEM, Athena over the bucket) — a log you can't search in a hurry isn't much use mid-incident.
5. **Alert on the log-tampering signals themselves**: `StopLogging`, `DeleteTrail`, `UpdateTrail`, bucket policy changes on the log bucket. If someone disables the trail, that action must page you.
6. **Test it.** Perform a known action and confirm it appears, correctly attributed, where you expect. An untested logging pipeline fails silently.

### Cheatsheet

```bash
aws cloudtrail describe-trails --query 'trailList[].{name:Name,region:IsMultiRegionTrail,val:LogFileValidationEnabled}'
aws cloudtrail get-trail-status --name TRAIL     # IsLogging: true?

aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin
aws cloudtrail lookup-events --lookup-attributes AttributeKey=Username,AttributeValue=<user>

StopLogging  DeleteTrail  UpdateTrail  PutBucketPolicy(on log bucket)
```

### Reading the setup

- **No trail, or single-region only** = blind spots; activity in unmonitored regions/accounts leaves no trace. Fix before anything else.
- **Management events only when you needed object access** = you'll know a bucket policy changed but not who read the objects — enable data events for the resources that matter.
- **Logs in an account/bucket the workload can modify** = the attacker who owns the workload owns the evidence. Isolate the log destination.
- **Log file validation off** = you can't prove the logs weren't altered — a problem for both investigation and compliance.
- **A `StopLogging` event with no alert** = the one event you most need to see got buried. Alerting on tampering is the backstop.

### The fix / best practice

- **One multi-region, all-account trail**, delivering to a **dedicated logging account** with a bucket the workloads can't write to or delete from.
- **Enable log file validation** and **Object Lock/versioning + MFA-delete** on the log bucket so history is immutable.
- **Turn on data events** selectively for high-value resources (sensitive S3, KMS, Lambda) — balance cost against the visibility you need.
- **Centralise and index** logs into a SIEM/queryable store, and **alert on tampering** (`StopLogging`/`DeleteTrail`) and other high-signal events.
- **Set retention** to meet investigation and compliance needs, and periodically **test** that a known action shows up correctly.

### Pitfalls

- **Assuming it's on by default.** Baseline logging often exists, but multi-region, data events, and tamper protection usually aren't configured until someone does it.
- **Log bucket in the same account with workload write access.** The attacker deletes the evidence of their own actions. Isolate it.
- **Skipping data events, then needing them.** You can't retroactively log object access that wasn't being recorded. Decide coverage in advance.
- **Collecting logs nobody can query.** During an incident, an unsearchable archive is nearly useless. Index it.

### References

- AWS CloudTrail best practices documentation
- AWS Security Reference Architecture (logging account pattern)
- Azure Activity Log / GCP Cloud Audit Logs equivalents
- NIST SP 800-92 (log management)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.