---
format: "v2"
name: "security-group-review"
title: "Security Group Review"
title_fr: "Revue des security groups"
description: "Use when auditing cloud network exposure — finding security groups and firewall rules that open sensitive ports to the world, and tightening them to least access."
description_fr: "À utiliser pour auditer l'exposition réseau dans le cloud — repérer les security groups et règles de pare-feu qui ouvrent des ports sensibles au monde entier, et les restreindre au strict nécessaire."
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

Security groups are the cloud's host firewall, and the classic mistake is a rule opening a sensitive port to `0.0.0.0/0` — a database, SSH, or RDP exposed to the entire internet. Attackers scan for exactly these. This skill covers auditing security-group rules for dangerous exposure and closing them to least access.

### When to use it

Cloud posture reviews, after infrastructure changes, and continuously (exposure creeps in with every quick "just open it to test" rule). Pairs with the network port-scanning skill — this is the cloud-native way to see exposure from the config side.

### Procedure

1. **Pull the rules and look for the world-open ones.** The dangerous pattern is a source of `0.0.0.0/0` (or `::/0`) on anything that isn't meant to be public:
   ```
   aws ec2 describe-security-groups \
     --query 'SecurityGroups[].{id:GroupId,perms:IpPermissions}' 
   ```
2. **Flag sensitive ports open to the internet** in priority order — these are the findings attackers act on fastest:
   - **22 (SSH), 3389 (RDP)** open to `0.0.0.0/0` — remote admin exposed; scanned and brute-forced constantly.
   - **Database ports** (3306, 5432, 1433, 27017, 6379, 9200) open to the world — direct data exposure.
   - **Internal/admin services** (management UIs, unauthenticated dashboards) exposed.
3. **Check what's actually attached.** A permissive rule on an unused security group matters less than the same rule on a group attached to a live, sensitive instance — prioritise by real exposure.
4. **Look for overly broad ranges**, not just `0.0.0.0/0` — a `/8` or the whole corporate range where a single host was intended is still over-exposure.
5. **Automate the sweep** across the account so you catch every region and group, not just the ones you remember:
   ```
   prowler aws --checks <security-group checks>
   ```
6. **Trace intent.** For each risky rule, determine what it's for — some public exposure is legitimate (a web server on 443). The finding is exposure that isn't justified.

### Cheatsheet

```bash
aws ec2 describe-security-groups --query \
 'SecurityGroups[?length(IpPermissions[?IpRanges[?CidrIp==`0.0.0.0/0`]])>`0`].GroupId'

22, 3389            -> SSH/RDP open to 0.0.0.0/0   (remote admin)
3306,5432,1433      -> SQL databases exposed
27017,6379,9200     -> Mongo/Redis/Elastic exposed (often unauthenticated!)
any admin/mgmt UI   -> exposed console

prowler aws          # includes many security-group exposure checks
```

### Reading the review

- **SSH/RDP open to `0.0.0.0/0`** = high finding; these are brute-forced within minutes of exposure. Restrict to a bastion or VPN range.
- **A database port open to the world** = critical, especially Redis/Mongo/Elasticsearch which historically shipped with no auth — internet exposure can mean direct data theft.
- **A permissive rule attached to a live sensitive instance** outranks the same rule on nothing — prioritise by what's actually reachable behind it.
- **`0.0.0.0/0` on 443 for a web server** = usually legitimate; not every world-open rule is a finding. Judge by whether the service is meant to be public.
- **Broad-but-not-world ranges** (`10.0.0.0/8` for one host) = over-provisioned access; tighten even if not internet-facing.

### The fix

- **Least access by default.** Open only the ports a resource needs, only to the sources that need them — a specific CIDR, a bastion, or another security group, not `0.0.0.0/0`.
- **No direct admin exposure.** SSH/RDP go through a bastion host or SSM/session manager (no inbound port at all), or at least restrict to a VPN range.
- **Databases never face the internet.** Put them in private subnets, reachable only from the app tier's security group.
- **Reference security groups, not IP ranges**, for internal service-to-service access — it's tighter and self-documenting.
- **Detect drift**: a scheduled check (Config rule / Prowler) that flags any new world-open sensitive port so exposure can't creep back in after a "temporary" rule.

### Pitfalls

- **"Temporary" open rules that stay.** The `0.0.0.0/0` added to debug never gets removed. Automated drift detection catches these.
- **Focusing on `0.0.0.0/0` only.** Overly broad private ranges are exposure too, just internal.
- **Ignoring what's attached.** Not every permissive group is live; prioritise by real reachability, but don't leave latent ones either.
- **Assuming a private subnet is enough.** A public-facing security group on an instance in a public subnet still exposes it; check both.

### References

- AWS VPC security group best practices
- CIS AWS Foundations Benchmark (network exposure controls)
- Prowler documentation
- CWE-284 (Improper Access Control)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.