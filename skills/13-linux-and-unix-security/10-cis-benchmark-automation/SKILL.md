---
name: cis-benchmark-automation
domain: 13-linux-and-unix-security
description: Use when applying and verifying a Linux hardening baseline at scale — automating CIS Benchmark checks and remediation so hosts start hardened and don't drift.
difficulty: intermediate
tags: [linux, cis, benchmark, hardening, automation, compliance]
tools: [openscap, ansible, lynis]
---

## Purpose

Hardening one host by hand is fine; hardening a fleet and keeping it hardened is a different problem. The CIS Benchmarks are a consensus baseline of Linux security settings — and the value comes from applying them consistently and *verifying* them continuously, not from a one-time manual pass. This skill covers automating the baseline so every host starts hardened and drift gets caught. It's the skill that ties the rest of the Linux domain together into something operable at scale.

## When to use it

Standing up a hardened Linux fleet, bringing an existing estate up to a baseline, or setting up continuous compliance checking. It operationalises the individual hardening skills (SSH, sudo, permissions, sysctl, auditd, SUID) into a repeatable, measurable standard.

## Procedure

1. **Pick the benchmark and level.** CIS Benchmarks come per-distro and with levels — Level 1 (sensible hardening with little operational impact) and Level 2 (stricter, for high-security environments, more likely to affect functionality). Start with Level 1 as the baseline for most fleets.
2. **Assess current state first — measure before changing.** Scan hosts against the benchmark to see where they stand and get a score. This baseline tells you what to fix and proves progress:
   ```
   # OpenSCAP against a CIS profile
   oscap xccdf eval --profile <cis-level1> --results out.xml <ssg-content>.xml
   # or a lighter audit
   lynis audit system
   ```
3. **Automate remediation with configuration management.** Apply the baseline through Ansible/Puppet/Chef (there are maintained CIS remediation roles) so it's consistent, repeatable, and version-controlled — not hand-applied per host. This is what makes it scale and what makes new hosts start hardened.
4. **Test the baseline against your workloads before fleet-wide rollout.** Some CIS settings (especially Level 2, and things like restrictive mount options or disabled protocols) can break legitimate applications. Apply to a canary, confirm apps work, and document any justified exceptions.
5. **Handle exceptions deliberately.** A setting that genuinely breaks a needed function gets a documented, risk-accepted exception — not a silent skip. The compliance report should show intended state vs actual with exceptions accounted for.
6. **Verify continuously and catch drift.** Re-scan on a schedule; hosts drift as people make changes, and a hardened host today isn't hardened next month. Alert on drift from the baseline and re-remediate.
7. **Feed the results into governance** — the compliance score and exceptions map to the GRC domain (audit evidence, risk register) and give leadership a measurable security posture.

## Cheatsheet

```bash
# assess (measure first) — pick a distro-specific CIS profile
oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis_level1 \
  --results results.xml ssg-<distro>-ds.xml
lynis audit system                 # lighter, quick posture audit

# remediate at scale (config management, version-controlled)
# maintained CIS hardening roles: ansible-lockdown, dev-sec/ansible-collection, etc.
ansible-playbook cis-hardening.yml -l canary     # canary FIRST

# then verify continuously
schedule the scan -> compare to baseline -> alert on drift -> re-remediate

workflow: MEASURE -> automate remediation -> test on canary -> handle exceptions
          -> continuous verify + drift alerting -> feed GRC
levels: L1 (default, low impact) | L2 (stricter, may break apps)
```

## Reading the results

- **A low compliance score across the fleet** = broad hardening gaps; the benchmark gives a prioritised, consensus list to work through, and a metric to track improvement.
- **Hosts hand-hardened inconsistently** = some secure, some not, and no way to know which; automation via config management is the fix — consistency is the point.
- **New hosts arriving unhardened** = the baseline isn't in the provisioning path; wire remediation into host build so hosts start compliant.
- **Drift since the last scan** = someone changed settings, or a deploy reverted them; this is why one-time hardening fails and continuous verification matters. Re-remediate.
- **Undocumented failures vs justified exceptions** = a failing check with no rationale is a gap; a documented risk-accepted exception is a decision. Distinguish them in the report.
- **High score, automated, continuously verified, with tracked exceptions** = the operable hardened state.

## The fix / best practice

- **Automate, don't hand-harden.** Apply the CIS baseline through version-controlled config management so it's consistent across the fleet and new hosts start hardened.
- **Measure before and after** with a scanner (OpenSCAP/Lynis) to prioritise and prove progress.
- **Canary-test before fleet rollout** — some settings break apps; validate, then document justified exceptions rather than skipping silently.
- **Verify continuously and alert on drift** — hardening is a state to maintain, not a one-time event.
- **Start at Level 1**, move to Level 2 where the security requirement justifies the operational cost.
- **Feed compliance data to GRC** for audit evidence and risk tracking.

## Pitfalls

- **One-time hardening.** Hosts drift; without continuous verification, the baseline erodes and you won't know. Re-scan and re-remediate on a schedule.
- **Hand-applying to a fleet.** Manual hardening is inconsistent and unrepeatable; automation is what makes it real at scale and keeps new hosts compliant.
- **Rolling out fleet-wide without a canary.** Some CIS settings break legitimate apps; a canary catches that before an outage.
- **Silent exceptions.** Skipping a failing check without documenting the reason looks identical to negligence in an audit. Record risk-accepted exceptions.
- **Chasing 100% blindly.** Some checks won't fit your environment; a documented, understood 90% beats a broken 100%. Tune deliberately.

## References

- CIS Benchmarks (cisecurity.org) — per-distro Linux baselines, Levels 1/2
- OpenSCAP / SCAP Security Guide, Lynis
- CIS hardening automation roles (ansible-lockdown, dev-sec)
- The other Linux hardening skills (this operationalises them) and the GRC domain
