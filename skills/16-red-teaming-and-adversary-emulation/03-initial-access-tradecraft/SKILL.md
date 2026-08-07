---
name: initial-access-tradecraft
domain: 16-red-teaming-and-adversary-emulation
description: Use when planning the initial-access phase of an authorised engagement — how adversaries get the first foothold (phishing, exposed services) and, for defenders, how to detect and prevent it.
difficulty: advanced
tags: [red-team, initial-access, phishing, foothold, authorized]
tools: []
---

## Purpose

Initial access is how an adversary gets their first foothold — and it's where most real intrusions begin, so emulating it tests the organisation's first line of defence. This skill covers the initial-access phase of an authorised engagement at the tradecraft-concept level: the main vectors adversaries use, how to emulate them within scope, and — the defensive payoff — how each is detected and prevented. It deliberately stays conceptual (no ready-to-fire payloads), consistent with the repo's scope.

## When to use it

The entry phase of a full-scope authorised engagement (assumed-breach engagements skip it). Always under a signed RoE (the scoping skill). The defensive framing is the point: understanding how footholds are gained is how the blue team learns to catch them.

## The main initial-access vectors (and their defence)

- **Phishing** — the dominant vector. A crafted email/message leads a user to run a payload, enter credentials, or authorise access. *Emulate:* an authorised phishing simulation within scope (the social-engineering domain covers the defensive side). *Detect/prevent:* email authentication (SPF/DKIM/DMARC), attachment/URL detonation, user reporting, and MFA (which defeats credential phishing).
- **Exposed/vulnerable services** — an internet-facing service with a vulnerability or weak authentication (the OSINT and vuln-management domains find these). *Emulate:* exploit an in-scope exposed service. *Detect/prevent:* attack-surface management, patching (especially KEV-listed), and strong authentication.
- **Valid accounts** — using credentials obtained from a breach/leak (the OSINT credential-leaks skill) or weak/default passwords. *Detect/prevent:* MFA, breached-password screening, anomalous-login detection.
- **Supply chain / third party** — compromise via a vendor or a malicious dependency (the supply-chain domain). *Detect/prevent:* third-party risk management, supply-chain controls.

## Procedure (authorised engagement)

1. **Work strictly within the RoE.** Initial-access tradecraft (especially phishing and exploiting services) directly affects real people and systems; confirm scope, permitted techniques, and stop conditions before anything.
2. **Reconnaissance first** (the OSINT domain) — the attack surface, exposed services, and people inform which initial-access vector is viable, mirroring how a real adversary starts.
3. **Choose the vector matching the emulated actor** (the emulation-planning skill) — emulate how *this actor* gains access, using the plan's ATT&CK-mapped approach.
4. **Emulate the access, focused on testing the defence.** The goal isn't to "win" but to test whether the organisation detects and prevents the foothold — did the phishing get reported and blocked? did the exposed service alert? did MFA stop the credential? The defensive outcome is the finding.
5. **Establish the foothold** if access succeeds, transitioning to the post-exploitation phases (C2, lateral movement) — but always noting what did and didn't get detected at initial access.
6. **Document what worked and what was detected** — for the blue team, "how the foothold was gained and whether it was caught" is a critical finding that drives improving the first line of defence.

## Cheatsheet

```
initial access = the FIRST foothold ; where most intrusions begin -> tests the first line of defence
  (conceptual — no ready-to-fire payloads ; authorised RoE only ; affects real people/systems)

main vectors + defence (the payoff)
  PHISHING (dominant)     emulate: authorised phishing sim ; detect/prevent: SPF/DKIM/DMARC,
                            detonation, user reporting, MFA (defeats cred phishing)
  EXPOSED/VULN SERVICES   emulate: exploit in-scope service ; detect/prevent: ASM, patching (KEV), strong auth
  VALID ACCOUNTS          leaked/weak creds ; detect/prevent: MFA, breached-pw screening, anomalous-login
  SUPPLY CHAIN/3RD PARTY  vendor/dependency ; detect/prevent: third-party risk, supply-chain controls

engagement: within RoE -> recon (OSINT) -> vector matching emulated actor -> emulate access
  goal = TEST THE DEFENCE (reported? alerted? MFA stopped it?) not "win"
  -> establish foothold (to post-exploitation) ; DOCUMENT what worked + what was DETECTED
```

## Reading the phase

- **Phishing succeeding and not being reported/blocked** = a gap in the first line of defence (email controls, user awareness, reporting culture); the finding drives improving those (the social-engineering domain). The dominant vector, so this matters most.
- **An exposed vulnerable service exploited for access** = an attack-surface and patching gap (OSINT/vuln-management); the foothold via an internet-facing service is a common real-world start.
- **Credential-based access succeeding without MFA stopping it** = the MFA gap; leaked/weak credentials plus no MFA is a top real intrusion path. MFA is what should have caught this.
- **Access gained but detected at initial access** = a defensive win to note; the blue team caught the foothold, which is exactly what you're testing. Detection at initial access is high-value.
- **How the foothold was gained and whether it was caught** = the critical finding; the initial-access phase's value is testing and improving the organisation's first line of defence, not the access itself.

## Pitfalls

- **Operating outside the RoE.** Initial access (phishing, exploiting services) affects real people and systems; confirm scope, permitted techniques, and stop conditions first. Non-negotiable.
- **Focusing on "winning" over testing the defence.** The value is whether the organisation detects and prevents the foothold; the defensive outcome (reported? alerted? MFA stopped it?) is the finding, not the access.
- **Emulating a vector the actor doesn't use.** Ground the choice in the emulation plan (how *this actor* gains access), not whatever's easiest.
- **Skipping the detection story.** "We got in" without "and here's what was/wasn't caught" wastes the engagement's defensive value; document the detection outcome.
- **Providing operational payloads.** This skill stays conceptual by design; the value is understanding vectors and their defences, not ready-to-fire attack tooling.

## References

- MITRE ATT&CK — TA0001 (Initial Access): T1566 (Phishing), T1190 (Exploit Public-Facing Application), T1078 (Valid Accounts), T1195 (Supply Chain)
- The scoping-and-rules-of-engagement, attack-emulation-planning skills
- The OSINT, social-engineering-defence, vulnerability-management, and supply-chain domains (the defences)
- PTES and NIST SP 800-115
