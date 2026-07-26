---
name: ad-enumeration-bloodhound
domain: 12-active-directory-and-windows-security
description: Use when you have a domain foothold and need to map Active Directory attack paths — collecting data with SharpHound and analysing it in BloodHound — plus what to fix.
difficulty: advanced
tags: [active-directory, bloodhound, enumeration, privilege-escalation, windows]
tools: [sharphound, bloodhound, ldapsearch]
---

## Purpose

Active Directory is a graph: users, groups, computers, and the permissions between them. Attackers win by finding a path of legitimate permissions from where they are to Domain Admin. BloodHound draws that graph so both sides can see it. This skill covers collecting the data, reading the paths, and cutting them.

## When to use it

After you've authenticated to a domain (any user, however low) on an authorised engagement, or when auditing your own AD for hidden escalation routes. It's the map that makes Kerberoasting, delegation abuse, and ACL attacks targeted instead of blind.

Domain-authenticated engagements only. Collection generates a lot of LDAP traffic and is itself detectable — expect it to show up in logs.

## Procedure

1. Collect the graph data from a domain-joined context with any valid credentials. `All` gathers sessions, ACLs, group membership, and trust:
   ```
   SharpHound.exe -c All
   # or the python collector, off-host:
   bloodhound-python -u user -p 'pass' -d domain.local -c All -ns <dc-ip>
   ```
2. Load the resulting JSON/zip into BloodHound and let it build the graph.
3. Run the built-in queries that matter first — they answer "how do I get to the top from here":
   - **Shortest paths to Domain Admins** from owned or low-priv principals.
   - **Kerberoastable users** (service accounts with SPNs) — feeds the Kerberoasting skill.
   - **AS-REP roastable users** (no pre-auth required).
   - **Dangerous ACLs**: `GenericAll`, `WriteDACL`, `WriteOwner`, `ForceChangePassword` over users/groups you can reach.
4. Mark what you actually control as "Owned" and re-query paths from those nodes — the realistic paths, not the theoretical ones.
5. Read each edge as an action: `MemberOf`, `AdminTo`, `HasSession`, `CanRDP`, `GenericAll`. The path is a recipe — each hop is a technique to execute (and, for defenders, to break).

## Cheatsheet

```
# collection
SharpHound.exe -c All --zipfilename loot
bloodhound-python -u U -p P -d dom.local -c All -ns 10.0.0.1

# high-value BloodHound queries
- Shortest Paths to Domain Admins
- Find Principals with DCSync Rights
- Kerberoastable Users
- AS-REP Roastable Users
- Shortest Path from Owned Principals

# raw LDAP sanity checks
ldapsearch -x -H ldap://dc -D 'user@dom' -w pass -b 'dc=dom,dc=local' '(servicePrincipalName=*)'
```

## Reading the output

- **A short path from your foothold to Domain Admins** is the headline finding — report the exact edges, because each is a fixable misconfiguration.
- **`DCSync` rights (`GetChanges`/`GetChangesAll`) on a non-DC principal** means that account can pull every hash in the domain. Critical.
- **`GenericAll`/`WriteDACL` over a privileged group** is a one-step escalation (add yourself, or reset a member's password).
- **Kerberoastable admins** — an SPN on a Domain Admin account is an offline-crack path straight to the top.
- **Unconstrained delegation on a computer** is a high-value target for the delegation-abuse skill.

## The fix

The paths are permissions, so remediation is tightening permissions:

- **Prune ACLs.** Remove `GenericAll`/`WriteDACL`/`WriteOwner` grants that aren't justified; these are usually leftovers nobody meant to leave.
- **Tier your admins.** Domain Admins should never log on to workstations — that's what creates the `HasSession` edges attackers hop through. Implement a tiered administration model.
- **Kill unnecessary SPNs** and give service accounts long, managed passwords (gMSA) so Kerberoasting yields nothing crackable.
- **Remove unconstrained delegation**; use constrained/resource-based delegation with tight scope.
- **Restrict DCSync rights** to actual domain controllers.
- Re-run collection after changes — the graph is the verification that a path is truly gone.

## Pitfalls

- **Collecting once and trusting it.** AD changes daily; a path closed last month may be back. Re-collect.
- **Chasing theoretical paths.** Filter to paths from principals you actually control — those are the real risk.
- **Forgetting collection is loud.** SharpHound is detectable; on a defensive audit that's fine, on a red-team plan for it.
- **Fixing a node, missing the edge.** Removing a user from a group doesn't help if a `WriteDACL` lets them re-add themselves. Fix the permission, not just the symptom.

## References

- BloodHound documentation (SpecterOps)
- MITRE ATT&CK — T1069 (Permission Groups Discovery), T1482 (Domain Trust Discovery)
- Microsoft — Securing privileged access / tiered administration model
