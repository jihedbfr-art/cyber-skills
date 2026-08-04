---
name: cspm-baseline
domain: 06-cloud-security
description: Use when setting up continuous cloud security posture monitoring — scanning cloud accounts against a baseline for misconfiguration, and keeping them from drifting back.
difficulty: intermediate
tags: [cloud, cspm, posture, misconfiguration, monitoring, compliance]
tools: [prowler, scoutsuite, cloudsploit]
---

## Purpose

Cloud misconfiguration isn't a one-time fix — accounts drift constantly as people add resources and "temporary" exceptions. Cloud Security Posture Management (CSPM) is the practice of continuously scanning your accounts against a baseline of good configuration, so a public bucket or an over-broad security group surfaces as a finding instead of a breach. This skill covers standing up posture monitoring with open tooling and keeping it useful rather than noisy.

## When to use it

Once you have more than a trivial cloud footprint, and especially across multiple accounts where manual review can't keep up. It ties together the individual cloud skills (S3, security groups, IAM, credentials, logging) into a continuous, measurable check rather than point-in-time audits.

## Procedure

1. **Run a baseline scan** across the account with an open CSPM tool to see where you stand. Prowler and ScoutSuite check hundreds of controls (many mapping to CIS benchmarks) and produce a scored report:
   ```
   prowler aws            # broad checks, CIS-aligned, per-account
   scoutsuite aws         # multi-service posture snapshot (HTML report)
   ```
2. **Map findings to the risks that matter.** The scan returns a lot; prioritise the high-impact classes this domain already covers — public storage, world-open sensitive ports, over-privileged IAM, root keys, disabled logging, unencrypted data. Don't drown the team in low-severity items.
3. **Establish the baseline you'll hold to** — pick a standard (CIS benchmark, or your own policy) as the definition of "good", so findings are measured against something explicit rather than opinion.
4. **Make it continuous, not one-off.** Schedule the scan (or use the provider's native posture service — Security Hub, Defender for Cloud, Security Command Center) so drift is caught as it happens. A one-time scan is stale within days.
5. **Route findings to owners and track them.** A CSPM report nobody acts on is noise; wire high-severity findings into tickets/alerts with owners, like the vulnerability-management workflow.
6. **Alert on drift for the critical controls** — a bucket going public, logging being disabled, a root key appearing — so the most dangerous regressions page someone immediately rather than waiting for the next scan.

## Cheatsheet

```bash
# baseline scan (open tools)
prowler aws                         # CIS-aligned, per-check pass/fail
prowler aws --severity critical high
scoutsuite aws                      # HTML posture report across services

# native alternatives (managed CSPM)
AWS Security Hub / Config  |  Azure Defender for Cloud  |  GCP Security Command Center

# prioritise these finding classes (high impact, covered by other skills)
public storage buckets | world-open sensitive ports | over-privileged IAM
root access keys | disabled/tampered logging | unencrypted sensitive data
no MFA on privileged users | overly permissive resource policies

make it CONTINUOUS + alert on critical DRIFT (bucket->public, logging off, root key)
```

## Reading the results

- **A cluster of high-severity findings** (public buckets, open admin ports, root keys) = the actionable core; work these first — they map directly to the breach scenarios the other cloud skills cover.
- **A low posture score dominated by low-severity items** = calibrate before dumping it on the team; leading with 500 lows buries the handful that matter. Filter by severity.
- **The same finding recurring after you fixed it** = drift; the fix didn't stick or a deploy reintroduced it — this is the signal that you need continuous monitoring and drift alerts, not just periodic scans.
- **Findings with no owner or ticket** = they won't get fixed; posture management is only real when findings flow into remediation.
- **A clean scan** = a point-in-time result with a short shelf life; schedule the next one and alert on critical drift, because clean today isn't clean next week.

## The fix / operating it well

- **Adopt a baseline** (CIS benchmark or documented policy) as the explicit definition of good, and scan against it.
- **Run continuously** — scheduled open-tool scans or a managed CSPM service — so drift surfaces quickly rather than accumulating.
- **Prioritise by real impact** (the high-severity classes above), not raw finding count, to keep the output actionable.
- **Route to owners and track to closure**, mirroring the vulnerability-management SLA approach.
- **Alert on critical drift** (public storage, logging disabled, root credentials, world-open ports) for immediate response.
- **Feed remediation back into IaC** so fixes are permanent and the next deploy doesn't reopen them.

## Pitfalls

- **One-time scanning.** Cloud drifts continuously; a single audit is stale fast. Make posture monitoring continuous.
- **Alert/finding overload.** Dumping every low-severity item makes the report ignored. Prioritise by impact and route only what matters.
- **Findings without owners.** A scored report that nobody acts on changes nothing. Wire findings into remediation with owners.
- **No drift alerting.** Even with periodic scans, a bucket going public between scans is a window of exposure. Alert on the critical regressions in real time.
- **Fixing in the console, not in IaC.** Manual fixes get overwritten by the next deploy; fix at the source so it holds.

## References

- Prowler, ScoutSuite, CloudSploit documentation
- CIS Benchmarks for AWS/Azure/GCP
- Provider CSPM services: AWS Security Hub/Config, Microsoft Defender for Cloud, GCP Security Command Center
- The other skills in this domain (each finding class in depth)
