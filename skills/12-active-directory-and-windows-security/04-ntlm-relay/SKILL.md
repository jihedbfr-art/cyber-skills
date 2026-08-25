---
format: "v2"
name: "ntlm-relay"
title: "Ntlm Relay"
title_fr: "Relais NTLM"
description: "Use when testing whether NTLM authentication can be relayed to authenticate to other services — a classic AD attack — and the signing/channel-binding that shuts it down."
description_fr: "À utiliser pour tester si une authentification NTLM peut être relayée vers d'autres services — une attaque AD classique — ainsi que la signature et le channel binding qui la neutralisent."
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

NTLM relay abuses the fact that NTLM authentication can be forwarded. An attacker who captures a victim's authentication attempt relays it to another service and authenticates *as the victim*, without ever knowing their password. Coerce a privileged account into authenticating and relay it to a sensitive service, and you can escalate across the domain. This skill covers the attack and the controls (signing, channel binding, disabling NTLM) that defeat it.

### When to use it

Internal AD engagements after you have network position. It pairs with coercion techniques (getting a machine or user to authenticate to you) and with BloodHound, which shows where a relayed identity gets you.

Authorised internal engagements only. Relaying real user authentication is high-impact — stay strictly within scope.

### Procedure

1. **Get in the authentication path.** Position so victims authenticate to you — via poisoning (LLMNR/NBT-NS with Responder) or IPv6 takeover (mitm6), or by coercing a target to authenticate to your host.
2. **Set up the relay.** Point captured authentication at a target service that accepts NTLM and lacks signing. `ntlmrelayx` does the relaying:
   ```
   ntlmrelayx.py -t ldaps://dc.domain.local -smb2support
   ```
3. **Trigger authentication** toward your relay — poisoning catches opportunistic auth; coercion (e.g. forcing a machine account to authenticate) produces high-value targets like domain controllers.
4. **Relay to a useful target.** Common high-impact relays: to **LDAP/LDAPS** on a DC (grant yourself rights, or with coercion set up delegation), to **SMB** on a host where the relayed account is admin (execute commands), or to **AD CS** web enrollment (request a certificate as the victim — the ESC8 attack).
5. **Confirm what the relayed identity unlocks** — command execution, a certificate you can auth with, or modified AD rights — and report the exact path.
6. Note whether targets **require signing / channel binding**; where they do, the relay fails — that's the defensive state you're validating.

### Cheatsheet

```bash
responder -I eth0                       # LLMNR/NBT-NS poisoning
mitm6 -d domain.local                    # IPv6 DNS takeover

ntlmrelayx.py -t ldaps://dc.domain.local -smb2support        # LDAP -> AD rights
ntlmrelayx.py -t smb://host --no-http-server -c 'whoami'      # SMB -> exec
ntlmrelayx.py -t http://ca/certsrv/certfnsh.asp --adcs        # AD CS (ESC8)

```

### Reading the output

- **A successful relay to SMB where the account is local admin** = command execution on that host as the victim. Direct compromise.
- **A relay to LDAP/LDAPS on a DC** with a privileged relayed identity = you can modify directory objects or set up delegation — a path to domain compromise.
- **A certificate issued via AD CS relay (ESC8)** = durable authentication as the victim, usable to escalate; high severity.
- **Relays failing because the target requires signing/channel binding** = the control is working — record which targets are protected and which aren't.
- **Machine-account coercion succeeding** is the difference between catching a random user and catching a domain controller — note the account relayed.

### The fix

Relay works when a service accepts NTLM without proving the channel is intact. Close that:

- **Enforce SMB signing** (required, not just enabled) everywhere — this kills SMB relay outright.
- **Enforce LDAP signing and LDAP channel binding** on domain controllers — stops the LDAP relay path.
- **Enable Extended Protection for Authentication (channel binding)** on HTTP services, especially **AD CS web enrollment** (or disable HTTP enrollment) to close ESC8.
- **Disable NTLM where possible** and move to Kerberos; where NTLM must stay, restrict it and monitor.
- **Stop the coercion/poisoning enablers**: disable LLMNR/NBT-NS, and disable IPv6 or protect DHCPv6 to blunt mitm6.
- Monitor for the poisoning and coercion signatures as a detection backstop.

### Pitfalls

- **Relaying back to the originating host.** Modern Windows blocks reflection; relay to a *different* target.
- **Chasing user auth when machine accounts are the prize.** Coercing a DC/machine account relays a far more powerful identity — plan for it.
- **Enabling signing but not requiring it.** "Enabled" still allows unsigned sessions; it must be *required* to stop the relay.
- **Fixing SMB, forgetting LDAP and AD CS.** Each relay target needs its own control; closing one leaves the others open.

### References

- MITRE ATT&CK — T1557 (Adversary-in-the-Middle), T1187 (Forced Authentication)
- Impacket ntlmrelayx documentation
- Microsoft guidance on SMB/LDAP signing and channel binding
- SpecterOps — AD CS relay (ESC8) research

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.