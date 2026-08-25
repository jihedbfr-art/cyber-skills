---
format: "v2"
name: "kerberoasting"
title: "Kerberoasting"
title_fr: "Kerberoasting"
description: "Use when you have any domain user and want to crack service-account passwords via their Kerberos service tickets — and how to make the attack yield nothing."
description_fr: "À utiliser dès qu'on dispose d'un compte de domaine, pour casser les mots de passe de comptes de service via leurs tickets Kerberos — et pour rendre cette attaque infructueuse."
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

Kerberoasting abuses a normal Kerberos feature: any domain user can request a service ticket for any account that has a Service Principal Name (SPN), and part of that ticket is encrypted with the service account's password hash. Pull the ticket, crack it offline, and you have the service account's plaintext password — often a privileged one. This skill covers the attack and the two settings that defeat it.

### When to use it

Right after domain foothold and BloodHound enumeration, which tells you which SPN accounts are worth roasting. It needs only a single valid domain account — no elevation — which is what makes it such a common early move.

Authorised, domain-joined engagements only. Requesting the tickets is low-noise but not invisible.

### Procedure

1. Find accounts with SPNs (the roastable set). BloodHound flags them, or query directly:
   ```
   # impacket, from Linux
   GetUserSPNs.py dom.local/user:pass -dc-ip <dc> -request
   ```
2. Request the service tickets and extract the crackable hashes. On Windows:
   ```
   Rubeus.exe kerberoast /outfile:hashes.txt
   ```
3. Prioritise. A ticket for a Domain Admin or a high-privilege service account is worth far more than a low-priv one — crack those first. BloodHound's "Kerberoastable users" with paths to DA tells you which.
4. Crack offline — this happens on your machine, so it's silent and unrate-limited:
   ```
   hashcat -m 13100 hashes.txt rockyou.txt
   ```
5. A cracked password is a real credential — use it within scope (log in, enumerate what it unlocks) and, crucially, report which account and how strong the password was, since that drives the fix.

### Cheatsheet

```bash
GetUserSPNs.py dom.local/user:pass -dc-ip 10.0.0.1 -request -outputfile hashes.txt

Rubeus.exe kerberoast /outfile:hashes.txt /nowrap

hashcat -m 13100 hashes.txt wordlist.txt -r rules/best64.rule

ldapsearch -x -H ldap://dc -D user@dom -w pass -b dc=dom,dc=local '(&(servicePrincipalName=*)(objectClass=user))'
```

### Reading the output

- **A ticket that cracks quickly** = a weak service-account password; the shorter/dictionary-based it is, the worse. That's the finding.
- **RC4-encrypted tickets (`$krb5tgs$23$`)** crack far faster than AES (`$krb5tgs$18$`) — their presence means downgrade-friendly config worth flagging.
- **A cracked Domain Admin or delegation-enabled service account** is critical: it's a direct path to domain compromise.
- **Tickets that resist cracking** (long random passwords / gMSA) are the goal state — note them as the accounts done right.

### The fix

Kerberoasting works when a service account has a weak, crackable password. Remove that:

- **Use Group Managed Service Accounts (gMSA)** or dedicated managed accounts. Their passwords are 120+ random characters, rotated automatically — uncrackable in practice. This is the real fix.
- Where a managed account isn't possible, enforce a **long (25+ char) random password** on the service account and rotate it.
- **Disable RC4** for Kerberos where you can, forcing AES, which is much slower to crack.
- **Remove unnecessary SPNs.** An account without an SPN can't be roasted.
- **Monitor for the behaviour**: a spike in TGS requests (event 4769), especially with RC4 encryption, from one user is a roasting signal. Feed it to the detection-engineering domain.

### Pitfalls

- **Roasting everything indiscriminately.** It's noisy at volume and most low-priv accounts aren't worth it. Target the SPN accounts with paths to privilege.
- **Assuming AES means safe.** AES is slower to crack, not impossible — a weak password still falls. Length is the real defence.
- **Fixing the password but leaving RC4 enabled.** The downgrade keeps cracking cheap. Address both.
- **Ignoring detection.** The request is legitimate Kerberos traffic, so prevention (strong passwords) matters more than hoping to block it — but the 4769 pattern is still worth alerting on.

### References

- MITRE ATT&CK — T1558.003 (Kerberoasting)
- Microsoft — Group Managed Service Accounts documentation
- Rubeus and Impacket documentation

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.