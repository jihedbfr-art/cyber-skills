---
name: dark-web-monitoring
domain: 21-threat-intelligence
description: Use when monitoring criminal forums, marketplaces, and leak sites for threats to your organisation — exposed credentials, data leaks, and chatter — done legally and safely.
difficulty: advanced
tags: [threat-intel, dark-web, monitoring, leaks, exposure]
tools: []
---

## Purpose

Criminal marketplaces, forums, paste sites, and ransomware leak sites are where stolen data is sold, breaches are announced, and attacks are discussed — sometimes before the target knows. Monitoring these sources can give early warning of exposed credentials, a data leak, an impending attack, or your organisation being discussed as a target. This skill covers dark-web and underground monitoring for threats to your organisation, done within legal and ethical bounds — the value is early warning, not participation.

## When to use it

Monitoring for external threats and exposure to your organisation, typically as part of a mature intel programme or via a specialist service. It complements the OSINT credential-leaks skill (public breach data) with the harder-to-reach underground sources.

## Procedure

1. **Define what you're monitoring for.** Focus on threats to *your* organisation: exposed/for-sale credentials, mentions of your company/brand/executives, leaked data attributed to you, ransomware leak-site listings, and chatter indicating you're a target. Monitoring "the dark web" generally is unfocused; monitor for your specific exposure.
2. **Stay within legal and ethical bounds — the critical constraint.** Monitoring and collecting intelligence is legitimate; *participating* in criminal activity is not. Don't purchase stolen data, don't engage in illegal transactions, don't hack, and don't impersonate to gain forum access in ways that cross legal lines. The line is observation vs participation — and it's easy to cross, so many organisations use specialist services or law enforcement channels rather than doing it directly.
3. **Prefer specialist services for direct access.** Dark-web monitoring is operationally hard and risky (access, attribution, safety, legality); commercial threat-intel providers with established, lawful access to these sources are how most organisations do it safely. Doing it in-house requires serious operational security and legal guidance.
4. **Monitor the high-value source types:** ransomware leak sites (are you listed?), credential markets and combolists (are your accounts for sale?), forums and paste sites (is your data or your name appearing?), and initial-access-broker listings (is access to your org being sold?).
5. **Vet and contextualise findings.** Underground data is often recycled, fake, or exaggerated — a "breach" for sale may be old, repackaged, or fabricated. Vet before acting (the IoC-vetting discipline): is the exposed data real, current, and actually yours?
6. **Act on confirmed exposure.** A confirmed finding drives concrete action: exposed credentials → force resets and MFA (the credential-leaks skill), a data leak → incident response and notification, an impending-attack signal → heightened defence and hunting. Early warning is only valuable if acted on.
7. **Handle findings sensitively.** Exposure intel is sensitive (it may reveal a breach you haven't disclosed); handle with appropriate confidentiality (TLP) and route to the right internal teams.

## Cheatsheet

```
underground = where stolen data is sold, breaches announced, attacks discussed
  value = EARLY WARNING of exposure/attack (not participation)

monitor for YOUR org (not "the dark web" generally)
  exposed/for-sale credentials | company/brand/exec mentions | leaked data attributed to you
  | ransomware leak-site listings | initial-access-broker listings | target chatter

CRITICAL: legal/ethical line = OBSERVATION vs PARTICIPATION
  OK: monitor, collect intel
  NOT OK: buy stolen data, illegal transactions, hacking, illegal access
  -> easy to cross -> most orgs use SPECIALIST SERVICES / law enforcement, not DIY
     (DIY needs serious opsec + legal guidance)

high-value sources: ransomware leak sites | credential markets/combolists | forums/pastes
  | initial-access brokers

VET findings (often recycled/fake/exaggerated) — real? current? actually yours?
ACT on confirmed: creds -> reset+MFA | leak -> IR+notify | attack signal -> heightened defence
handle SENSITIVELY (TLP — may reveal undisclosed breach)
```

## Reading the monitoring

- **Your organisation on a ransomware leak site** = a serious, time-sensitive finding — often the first sign of a breach, or a threat to publish stolen data. Immediate IR and decision-making; this is among the highest-value early warnings.
- **Credentials for your domain for sale** = an exposure to act on (force resets, MFA); but vet first — combolists recycle old, already-changed credentials, so confirm they're real and current before scrambling.
- **A "breach" of your data offered for sale** = vet carefully; underground data is frequently fake, recycled, or exaggerated to make a sale. Confirm it's real, current, and actually yours before treating it as a breach.
- **Initial-access-broker listing selling access to your org** = a strong pre-attack warning; someone may already be in or about to be. Trigger hunting and heightened defence.
- **The observation/participation line being approached** = stop; buying data or engaging in transactions crosses into criminal activity regardless of defensive intent. Use specialist services or law enforcement for anything near the line.
- **Confirmed, vetted exposure routed to the right teams and acted on** = the value realised; early warning that drives resets, IR, or heightened defence.

## Pitfalls

- **Crossing the observation/participation line.** Buying stolen data, engaging in illegal transactions, or illegal access is criminal even with defensive intent — and easy to drift into. Stay on the observation side; use specialist services or law enforcement for direct access.
- **Doing it in-house without opsec/legal guidance.** Direct dark-web access is operationally risky (safety, attribution, legality); most organisations should use commercial providers with lawful access rather than DIY.
- **Not vetting findings.** Underground data is often recycled, fake, or exaggerated; acting on an unverified "breach" wastes effort or causes panic. Vet before acting.
- **Monitoring generally instead of for your exposure.** "The dark web" is vast; focus on threats to your specific organisation, not undirected browsing.
- **Not acting on confirmed findings.** Early warning is only valuable if it drives resets, IR, or defence; findings that sit unactioned waste the whole capability.
- **Mishandling sensitive findings.** Exposure intel may reveal an undisclosed breach; handle confidentially and route appropriately.

## References

- The OSINT email-and-credential-leaks and IR skills
- Commercial dark-web / digital-risk-protection services
- Legal guidance on threat-intelligence collection boundaries (jurisdiction-specific)
- The ioc-collection-and-vetting skill (vetting discipline) and TLP handling
