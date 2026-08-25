---
format: "v2"
name: "scoping-and-rules-of-engagement"
title: "Scoping And Rules Of Engagement"
title_fr: "Cadrage et règles d'engagement"
description: "Use before any offensive engagement to define scope, authorisation, and boundaries in writing — the document that separates authorised testing from a crime."
description_fr: "À utiliser avant tout engagement offensif pour définir par écrit le périmètre, l'autorisation et les limites — le document qui distingue un test autorisé d'un acte délictueux."
domain: "16-red-teaming-and-adversary-emulation"
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

Everything else in this domain is illegal without this. The rules of engagement (RoE) are the written agreement that defines what you may test, how, when, and with whose authority. This skill covers building that document so the engagement is authorised, safe, and defensible — for the client and for you.

There's no technical payload here. That's the point: the most important step in offensive security is paperwork, and skipping it turns a professional engagement into unauthorised access.

### When to use it

Before a single packet is sent, on every engagement — pentest, red team, purple team. Read this before any other skill in this domain. If the RoE isn't signed, the engagement hasn't started.

### Procedure

1. **Get written authorisation from someone empowered to give it.** Not a verbal OK, not an email from a friendly engineer — a signed authorisation from a party who actually owns or controls the systems and has authority to permit testing. This is your legal shield.
2. **Define scope precisely.** List exactly which IP ranges, domains, applications, accounts, and physical locations are in — and explicitly what's **out**. Ambiguity here is where engagements go wrong. Third-party/cloud assets may need the provider's authorisation too, since you don't own their infrastructure.
3. **Define the rules of engagement:**
   - Permitted techniques and explicitly forbidden ones (e.g. no DoS, no destructive actions, no real data exfiltration — use markers).
   - Testing windows (dates, and time-of-day limits to avoid business-critical periods).
   - Social engineering: allowed or not, and against whom (never personal accounts/devices unless explicitly authorised).
   - Data handling: how findings and any accessed data are stored, transmitted, and destroyed.
4. **Set up deconfliction and emergency contacts.** A 24/7 point of contact on both sides, a way to prove "this traffic is us" if the blue team or a provider reacts, and a defined stop condition. Agree what happens if you find an active, pre-existing breach.
5. **Agree the objectives and the model.** Goal-based (reach specific data/systems) vs coverage-based; assumed-breach vs external; how much the defenders know. This shapes everything downstream.
6. **Get sign-off, distribute, and keep it accessible.** Every operator carries the authorisation and scope. Re-confirm before starting if anything changed.

### Cheatsheet

```
RoE must-haves
  [ ] signed authorisation from a party empowered to grant it
  [ ] exact in-scope assets (IPs, domains, apps, accounts, sites)
  [ ] explicit out-of-scope list
  [ ] permitted vs forbidden techniques (DoS? destructive? exfil?)
  [ ] testing windows + blackout periods
  [ ] social engineering: allowed? targets? exclusions?
  [ ] third-party/cloud provider authorisation where needed
  [ ] data handling + destruction terms
  [ ] emergency contacts (both sides, 24/7) + deconfliction
  [ ] stop conditions + "what if we find a real breach"
  [ ] objectives + engagement model (assumed-breach? goal-based?)

golden rule: if it's not in writing and signed, it's not authorised.
```

### Reading a proposed scope

- **A verbal-only or informal authorisation** is not sufficient — stop and get it in writing from someone with authority. This protects you as much as the client.
- **"Test everything" with no boundaries** is a red flag, not a green light — it usually means the signer hasn't thought about out-of-scope systems, shared infrastructure, or third parties they can't authorise.
- **Cloud/SaaS assets in scope** may require the provider's own authorisation; the client can't always grant access to infrastructure they rent.
- **No emergency contact or stop condition** means an incident (an outage you caused, a real breach you found) has no safe path — insist on it before starting.

### Why it matters (the substance)

- **Legal protection.** Authorised testing and criminal unauthorised access differ by exactly this document. It's the record that you had permission.
- **Safety.** Boundaries and stop conditions prevent an engagement from taking down production or straying into systems that aren't the client's to offer.
- **Trust and value.** Clear objectives and deconfliction let the blue team learn from the exercise instead of firefighting a mystery, which is the actual point of red teaming.

### Pitfalls

- **Starting on a verbal go-ahead.** The single most dangerous shortcut in offensive security. Written, signed, from an authorised party — every time.
- **Fuzzy scope.** "The main app" turns into "wait, that shared host also runs another client" mid-engagement. Enumerate exact assets and exclusions.
- **Ignoring third-party authorisation.** The client owning an app doesn't mean you're cleared to hammer the cloud provider's infrastructure under it.
- **No stop condition or breach plan.** When you cause an outage or find a live intruder, "what now" must already be answered.

### References

- PTES (Penetration Testing Execution Standard) — Pre-engagement Interactions
- NIST SP 800-115 (Technical Guide to Information Security Testing)
- MITRE ATT&CK (for structuring the emulation objectives that scope enables)
- SANS — Rules of Engagement guidance

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.