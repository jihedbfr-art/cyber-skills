---
name: lsass-protection
domain: 12-active-directory-and-windows-security
description: Use when hardening Windows hosts against credential theft from LSASS memory — Credential Guard, LSA protection, and the settings that make dumping tools come back empty.
difficulty: intermediate
tags: [active-directory, lsass, credential-guard, windows, hardening]
tools: []
---

## Purpose

LSASS is the Windows process that holds credentials of logged-on users in memory — which is exactly why every credential-dumping tool goes for it. This skill is the defensive counterpart to the credential-dumping skill: making LSASS a dead end, so that even an attacker with local admin walks away with little worth cracking. It's one of the highest-leverage host hardening steps in an AD environment.

## When to use it

Hardening Windows workstations and servers, especially anywhere privileged accounts log on. It pairs with the tiered-admin model (which keeps high-tier credentials off low-trust hosts) and the credential-dumping skill (the attack it defeats). Deploy it broadly via GPO.

## The protections

- **Credential Guard** — uses virtualization-based security (VBS) to isolate secrets (NTLM hashes, Kerberos TGTs) in a hardware-protected container that even a SYSTEM-level process can't read. This is the strongest control: dumping LSASS no longer yields those secrets.
- **LSA Protection (RunAsPPL)** — marks LSASS as a protected process, so non-protected processes (including most dumping tools) can't open its memory. Weaker than Credential Guard and bypassable by a determined attacker with a driver, but a solid, easy layer.
- **WDigest disabled** — legacy WDigest stored credentials in *cleartext* in LSASS. It should be off (default on modern Windows), but verify — a single enabled WDigest setting hands the attacker plaintext passwords.
- **Attack-surface reduction / EDR** — blocking known LSASS-access behaviours as defence in depth.

## Procedure

1. **Verify WDigest is disabled** first — it's the cheapest, highest-impact check. If `UseLogonCredential` is enabled anywhere, LSASS holds cleartext passwords; turn it off. On modern Windows it's off by default, but legacy configs and re-enablements happen.
2. **Enable LSA Protection (RunAsPPL)** across hosts — it blocks the common dumping tools with minimal compatibility impact. Set the registry/GPO value and confirm LSASS runs protected.
3. **Deploy Credential Guard** where the hardware supports it (VBS requirements) — this is the real fix, isolating secrets so LSASS dumps come back empty. Validate compatibility (it affects some legacy auth like unconstrained delegation and older protocols) before broad rollout.
4. **Layer EDR / attack-surface-reduction rules** that detect or block processes opening LSASS handles — catches the techniques that try to work around the above.
5. **Verify it works** — from an authorised test, attempt a dump and confirm the protections deny access or return no usable secrets. A protection you haven't verified is an assumption.
6. **Deploy at scale via GPO** and monitor for LSASS-access attempts as a detection signal.

## Cheatsheet

```
protections (strongest first)
  Credential Guard (VBS)   isolates secrets in hardware-protected container
                           -> LSASS dump yields nothing usable  (the real fix)
  LSA Protection (PPL)     LSASS = protected process, blocks common dumpers
                           -> easy layer, bypassable by driver
  WDigest disabled         no CLEARTEXT creds in LSASS  (verify — legacy leak)
  EDR / ASR                block/alert on processes opening LSASS handles

deploy order
  1. verify WDigest off (UseLogonCredential = 0)         <- cheapest, critical
  2. enable LSA Protection (RunAsPPL) via GPO
  3. roll out Credential Guard where VBS supported (test compat first)
  4. EDR/ASR rules on LSASS access + monitor
  5. TEST: attempt a dump -> confirm denied / empty
```

## Reading the state

- **WDigest enabled anywhere** = LSASS holds cleartext passwords; a dump gives the attacker plaintext, no cracking needed. The most urgent thing to fix and the easiest to miss.
- **No LSA Protection** = the standard dumping tools work unimpeded on any host where the attacker has admin. Enabling PPL blocks most of them cheaply.
- **Credential Guard deployed and verified** = the strong state; even a SYSTEM-level dump doesn't yield the isolated secrets. This is what turns credential theft from "expected" to "hard".
- **PPL/Credential Guard configured but never tested** = an assumption; misconfiguration or a compatibility fallback may leave it inactive. Verify with an actual dump attempt.
- **Protections present but privileged accounts still logging into these hosts** = defence in depth working, but combine with tiering — the best outcome is no high-tier credential on the host *and* LSASS hardened.

## Pitfalls

- **Forgetting WDigest.** Teams deploy Credential Guard but overlook a legacy WDigest setting that still dumps cleartext. Verify it's off.
- **Assuming LSA Protection is enough.** PPL is bypassable by an attacker who can load a driver; it's a layer, not the whole answer. Credential Guard is the stronger control.
- **Not testing Credential Guard compatibility.** It breaks some legacy scenarios (unconstrained delegation, older auth); test before broad rollout, but don't let that delay it indefinitely.
- **Deploying without verifying.** Config that silently fell back to unprotected is common. Confirm with a dump attempt on a test host.
- **Hardening LSASS but ignoring tiering.** The strongest posture is both — keep privileged credentials off the host (tiering) *and* protect LSASS, so a compromise yields nothing either way.

## References

- Microsoft — Windows Defender Credential Guard documentation
- Microsoft — Configuring Additional LSA Protection (RunAsPPL)
- MITRE ATT&CK — T1003.001 (LSASS Memory) as the defended technique
- Microsoft security baselines (deliver these via GPO)
