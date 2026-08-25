---
format: "v2"
name: "takedown-workflow"
title: "Takedown Workflow"
title_fr: "Procédure de retrait (takedown)"
description: "Use when getting malicious lookalike sites, phishing pages, and impersonation content removed — the process for reporting and taking down infrastructure that targets your organisation."
description_fr: "À utiliser pour faire retirer les sites frauduleux imitant votre marque, les pages de phishing et les contenus d'usurpation — la procédure de signalement et de retrait de l'infrastructure malveillante qui cible votre organisation."
domain: "17-social-engineering-defence"
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

When attackers set up a phishing page impersonating your login, register a lookalike domain, or create fake social profiles of your brand/executives, taking that infrastructure down protects your users and customers from being phished through it. Takedown is the process of getting malicious content removed by the parties who can remove it — registrars, hosts, platforms. This skill covers running a takedown workflow, the response that shrinks the attacker's reach against your organisation.

### When to use it

When you discover malicious infrastructure targeting your organisation — from phishing analysis (a harvest page), certificate-transparency/domain monitoring (a lookalike domain), or dark-web/brand monitoring (impersonation content). It's the action arm of external threat detection.

### Procedure

1. **Identify who can take it down.** Different infrastructure has different responsible parties, and knowing which to contact is half the battle:
   - **A phishing page** — the hosting provider (they can remove the content) and the domain registrar.
   - **A lookalike/malicious domain** — the registrar (for removal) and, faster, browser/email blocklists (Google Safe Browsing, Microsoft SmartScreen) which flag it in browsers even before removal.
   - **Fake social profiles / brand impersonation** — the platform (each has an impersonation/abuse reporting process).
   - **Malicious content in search/ads** — the search engine or ad platform.
2. **Gather evidence.** A takedown request needs proof: the malicious URL, screenshots of the impersonation/phishing, evidence it's harming your users, and (for brand cases) proof of your trademark/ownership. Well-documented requests get actioned faster.
3. **Report through the right channels.** Submit to the responsible party's abuse/takedown process — registrar/host abuse contacts, platform impersonation reporting, and browser/email blocklists. Blocklisting is often the fastest protection (it flags the site to users quickly, even before the host removes it), so do it in parallel with the removal request.
4. **Use takedown services for scale.** For organisations facing frequent impersonation, commercial brand-protection/takedown services (and some threat-intel providers) handle detection and takedown at scale with established relationships that speed removal. Doing it in-house works but is slower per-incident.
5. **Blocklist in parallel — protect users immediately.** Removal can take time (hours to days depending on the party); meanwhile, getting the URL onto browser/email blocklists and into your own email/web filtering protects your users *now*. Immediate protection matters more than waiting for removal.
6. **Preserve evidence before it disappears.** Capture the malicious content before takedown (screenshots, the page source, IoCs) — for the record, for the investigation, and because it may vanish once reported (the phishing-analysis and IoC skills).
7. **Track and follow up.** Takedowns aren't always immediate; track requests, follow up if not actioned, and escalate (some registrars/hosts are slow or unresponsive). For persistent attackers, involve law enforcement.

### Cheatsheet

```
attacker infra targeting you (phishing page / lookalike domain / fake profile) -> TAKE IT DOWN
  = get it removed by who CAN remove it (registrar/host/platform) -> shrinks attacker reach

WHO can take it down (knowing this = half the battle)
  phishing page      -> HOSTING provider + registrar
  lookalike domain   -> registrar + BROWSER/EMAIL BLOCKLISTS (Safe Browsing/SmartScreen — faster)
  fake social/brand  -> the PLATFORM (impersonation/abuse process)
  malicious search/ads -> search engine / ad platform

process
  GATHER EVIDENCE (URL, screenshots, harm, trademark/ownership) -> faster action
  REPORT to the right channel (abuse contacts / platform reporting / blocklists)
  BLOCKLIST IN PARALLEL -> protects users NOW (removal takes hours-days) — immediate > waiting
  takedown SERVICES for scale (frequent impersonation — established relationships speed it)
  PRESERVE evidence before it disappears (may vanish once reported)
  TRACK + follow up + escalate (slow/unresponsive parties ; law enforcement for persistent)
```

### Reading the workflow

- **Knowing the right party to contact** (host for content, registrar for domain, platform for profiles, blocklists for browser protection) = half the battle; a takedown request to the wrong party stalls. Match the infrastructure to who can remove it.
- **Blocklisting done in parallel with removal** = users protected immediately; removal can take hours to days, but blocklisting flags the site in browsers/email fast. Waiting only for removal leaves users exposed meanwhile — immediate protection is the priority.
- **A well-documented request (URL, screenshots, harm, ownership)** = actioned faster; sparse requests get deprioritised. Evidence quality drives takedown speed.
- **Frequent impersonation handled in-house** = slow per-incident; takedown services with established provider relationships scale better for organisations facing many attacks.
- **Evidence not preserved before takedown** = the content may vanish once reported, losing it for investigation and record. Capture it first.
- **Slow/unresponsive registrars or hosts** = expected sometimes; track, follow up, escalate, and involve law enforcement for persistent attackers. Takedown isn't always immediate.

### Pitfalls

- **Waiting only for removal.** Takedown takes time; blocklist in parallel to protect users immediately. Immediate protection matters more than waiting for the host to act.
- **Contacting the wrong party.** A request to someone who can't remove the content stalls; match the infrastructure (host/registrar/platform/blocklist) to who's responsible.
- **Sparse takedown requests.** Poorly-documented requests get deprioritised; provide the URL, screenshots, evidence of harm, and ownership proof for faster action.
- **Not preserving evidence.** Content may disappear once reported; capture it first for the record and investigation.
- **No follow-up.** Takedowns aren't always immediate or successful; track requests, follow up, and escalate slow/unresponsive parties.
- **Doing high-volume takedowns manually.** For frequent impersonation, in-house is slow; takedown services scale with established relationships.

### References

- Google Safe Browsing, Microsoft SmartScreen, and APWG reporting (blocklisting)
- The phishing-email-analysis, certificate-transparency, and dark-web-monitoring skills (detection sources)
- Registrar/host abuse-reporting processes and platform impersonation-reporting docs
- Commercial brand-protection / anti-phishing takedown services

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.