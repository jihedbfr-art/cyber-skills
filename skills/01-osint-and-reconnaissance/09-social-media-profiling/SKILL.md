---
format: "v2"
name: "social-media-profiling"
title: "Social Media Profiling"
title_fr: "Profilage sur les réseaux sociaux"
description: "Use when mapping an organisation's people and technology from public social and professional profiles — the human attack surface for phishing and pretexting — done within ethical and legal bounds."
description_fr: "À utiliser pour cartographier les collaborateurs et technologies d'une organisation à partir de profils sociaux et professionnels publics — la surface d'attaque humaine exploitée par le phishing et le pretexting — dans un cadre strictement éthique et légal."
domain: "01-osint-and-reconnaissance"
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

Attackers target people, and people publish a lot. Public professional and social profiles reveal an organisation's staff, roles, reporting lines, technologies they work with, and details useful for pretexting. This skill covers building that picture from public sources to understand the human attack surface — primarily so defenders know what an attacker can learn and can prepare people accordingly. It stays within public, ethical, legal bounds: public profiles only, no deception to extract private data.

### When to use it

External recon to understand who works at an organisation and what an attacker could use for social engineering, or (defensively) to assess your own exposure and inform awareness training. It feeds the phishing-defence domain: you can't defend against pretexts you haven't anticipated.

Keep it ethical and legal: gather only public information, don't create fake profiles to connect with or deceive employees, and respect privacy and platform terms. The goal is understanding exposure, not stalking individuals.

### Procedure

1. **Enumerate the organisation's people** from professional networks and the company site. Roles and departments reveal structure — who's in IT, finance, HR, leadership (the common phishing/BEC targets):
   ```
   theHarvester -d example.com -b linkedin,bing      # emails + names from public sources
   ```
2. **Derive the email/username convention** by combining names with the pattern you found in metadata/breach data (`first.last@`, `flast@`) — this turns a staff list into a target list.
3. **Map the technology and tooling** staff mention — job posts, profile skills, and "we use X" posts reveal the internal stack (which VPN, which cloud, which security tools), useful for both targeting and tailored pretexts.
4. **Identify high-value pretexting details** — an attacker crafts believable phishing from public specifics: a recent company event, an org-chart relationship (finance reports to this CFO), a vendor relationship, an ongoing project. Note what's exposed, not to exploit individuals but to see what makes a convincing lure.
5. **Assess leadership and finance exposure specifically** — these are the BEC/whaling targets; their public footprint (and their assistants') is worth understanding.
6. **Aggregate into a picture**, not a dossier on individuals — the deliverable is "here's the human attack surface and the pretexts it enables", to drive defences.

### Cheatsheet

```bash
theHarvester -d example.com -b bing,linkedin,duckduckgo

staff + roles + departments        -> who to target (IT/finance/HR/leadership)
email/username convention           -> staff list becomes target list
technologies mentioned              -> internal stack (VPN, cloud, tools)
pretexting hooks                    -> events, projects, vendor/org relationships
leadership + assistants             -> BEC / whaling exposure

ethics/legal boundary
  public info only | no fake profiles | no deception to extract private data
  goal = understand exposure to DEFEND, not to stalk or exploit individuals
```

### Reading the output

- **A complete staff list plus the email convention** = a ready-made phishing target list; this combination is the core human-attack-surface finding. Defensively, it's what your awareness training must assume attackers have.
- **Technology and tooling disclosed by staff** = tailored-pretext fuel ("IT is migrating to X, click here to re-authenticate") and targeting for the real stack. Flag what's oversharing.
- **Rich leadership/finance footprints** = elevated BEC/whaling risk; these roles and their assistants warrant specific awareness prep.
- **Pretexting hooks** (public events, projects, vendor relationships) = the specifics that make phishing believable — knowing them lets defenders inoculate against those exact lures.
- **Minimal public exposure** = a lower human attack surface; note it as a relative strength.

### The fix / defensive use

- **Feed it into awareness training** — use the real pretexts an attacker could build (from your actual public footprint) in phishing simulations, so training matches the threat (ties into the social-engineering-defence domain).
- **Guide staff on oversharing** — especially disclosing internal technology, security tooling, and project details publicly. Awareness, not prohibition; people can share professionally without handing attackers a stack diagram.
- **Prepare high-value targets** — leadership, finance, and their assistants get focused training on BEC/whaling, since their public exposure makes them prime targets.
- **Harden the technical layer regardless** — since you can't remove people's public presence, pair awareness with MFA, email authentication (SPF/DKIM/DMARC), and BEC detection, so a convincing pretext still fails.

### Pitfalls

- **Crossing the ethical/legal line.** Creating fake profiles, deceiving employees to extract private info, or scraping in violation of terms turns recon into something else. Public information only, for defensive understanding.
- **Building dossiers on individuals.** The goal is organisational attack surface and pretexts, not surveilling people. Keep it aggregate and purpose-bound.
- **Treating it as low-value.** Social engineering is one of the most successful attack vectors; understanding the human surface is exactly how you defend it.
- **Stopping at "here's the exposure" without acting.** The value is feeding it into training and technical controls — an unactioned profile of your weaknesses helps only the attacker.

### References

- OWASP WSTG-INFO — collect information about the target
- theHarvester documentation
- OSINT framework (osintframework.com) for people-search sources
- The social-engineering-defence domain (where this recon feeds the defences)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.