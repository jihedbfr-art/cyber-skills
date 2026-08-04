---
name: ad-hardening-baseline
domain: 12-active-directory-and-windows-security
description: Use when you need a prioritised checklist of the Active Directory settings that shut most attack paths — the baseline that turns a soft domain into a hard target.
difficulty: intermediate
tags: [active-directory, hardening, baseline, windows, defense]
tools: [pingcastle, purpleknight]
---

## Purpose

The individual AD skills each attack or defend one thing; this one steps back and gives the prioritised baseline — the settings that, together, close the paths behind most real-world domain compromises. It's the checklist to bring a neglected domain up to a defensible standard, and to measure where an existing one stands. Think of it as the index that ties the domain together.

## When to use it

Hardening an AD environment from scratch, doing a posture assessment, or after an engagement to make sure the systemic issues (not just the findings) get closed. Run an automated AD health tool first to see where you stand, then work the baseline.

## Procedure

1. **Measure first.** Run an AD security-posture scanner (PingCastle, Purple Knight) to get a scored baseline of your current state — it surfaces most of the issues below and gives you a before/after number to track. Don't harden blind.
2. **Work the baseline in priority order** — these are the high-leverage items, each with its own skill for the detail:
   - **Kill credential-theft escalation:** implement the **tiered-admin model** and protect **LSASS** (Credential Guard / LSA Protection, WDigest off). This closes the most common escalation chain.
   - **Fix delegation:** remove **unconstrained delegation**, mark privileged accounts sensitive/Protected Users, scope constrained/RBCD tightly.
   - **Harden authentication protocols:** enforce **SMB signing** and **LDAP signing + channel binding** (kills NTLM relay), disable NTLMv1 and legacy protocols, disable LLMNR/NBT-NS.
   - **Close roasting exposure:** require Kerberos **pre-authentication** everywhere (AS-REP), move service accounts to **gMSA** and remove unneeded SPNs (Kerberoasting), disable RC4 where possible.
   - **Lock down privileged access:** minimise Domain Admins, use **LAPS** for unique local admin passwords, restrict who holds replication rights (DCSync), and audit dangerous ACLs and GPO edit rights.
   - **Baseline the endpoints:** deploy Microsoft security baselines via GPO, patch DCs and hosts, restrict local admin.
3. **Verify with the graph.** Re-run BloodHound and confirm the short paths from workstations to Domain Admins are gone — the map is the proof that the baseline actually closed the paths, not just the settings.
4. **Set up detection** for the attacks the baseline is meant to prevent (roasting, DCSync, relay, delegation abuse) so a bypass is visible.
5. **Re-scan and track.** Re-run the posture tool, compare the score, and schedule periodic re-assessment — AD drifts back toward insecure defaults over time.

## Cheatsheet

```
measure:  PingCastle / Purple Knight  -> scored baseline (before/after)

baseline, by priority (each has its own skill)
  [ ] tiered-admin model + PAWs           (break credential-theft escalation)
  [ ] LSASS: Credential Guard, LSA prot., WDigest OFF
  [ ] remove unconstrained delegation; Protected Users for admins
  [ ] SMB signing + LDAP signing/channel binding  (kill NTLM relay)
  [ ] disable NTLMv1, LLMNR/NBT-NS, legacy protocols
  [ ] require Kerberos pre-auth (AS-REP); gMSA + fewer SPNs (Kerberoast); RC4 off
  [ ] minimise Domain Admins; LAPS; restrict replication rights (DCSync)
  [ ] audit dangerous ACLs + GPO edit rights (tier them)
  [ ] Microsoft security baselines via GPO; patch DCs/hosts

verify:  BloodHound -> short paths to DA gone?   detection on roast/DCSync/relay
track:   re-scan, compare score, re-assess on a cadence
```

## Reading the posture

- **Short BloodHound paths from a normal workstation to Domain Admins** = the domain is soft regardless of individual settings; the tiering + LSASS + delegation items are where to start, because they close those paths.
- **A high PingCastle/Purple Knight risk score** = concrete, prioritised issues to work; use it as the roadmap and the metric, not just a report.
- **Legacy protocols still enabled** (NTLMv1, unsigned SMB/LDAP, LLMNR) = relay and downgrade attacks remain open; these are usually quick, high-impact wins.
- **Service accounts with SPNs and weak passwords, or accounts with pre-auth off** = roasting exposure; gMSA and requiring pre-auth close it.
- **Score improved but BloodHound paths unchanged** = you fixed settings that weren't the actual escalation path; the graph is the truer measure — chase the paths, not just the checklist.

## Pitfalls

- **Hardening blind.** Without measuring first (a posture scan, BloodHound), you can't prioritise or prove progress. Baseline, then improve, then re-measure.
- **Chasing settings instead of paths.** A better config score with the same attack paths open isn't real progress; verify with the graph that the escalation routes are gone.
- **Doing it once.** AD drifts back toward insecure defaults as people add exceptions and legacy systems. Re-assess on a cadence.
- **Skipping the hard structural items.** Tiering and delegation cleanup are the highest-impact and the most work, so they get deferred — but they're exactly what closes the common compromise paths. Don't stop at the easy protocol toggles.
- **Baseline without detection.** Prevention will eventually be bypassed; pair the hardening with detection for the attacks it targets.

## References

- Microsoft — Securing Privileged Access / security baselines
- PingCastle and Purple Knight (AD posture assessment)
- CISA and CIS Active Directory hardening guidance
- The other skills in this domain (each baseline item in depth); BloodHound for verification
