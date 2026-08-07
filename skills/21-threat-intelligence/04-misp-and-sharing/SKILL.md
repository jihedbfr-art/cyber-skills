---
name: misp-and-sharing
domain: 21-threat-intelligence
description: Use when running a threat-intel sharing platform and exchanging intelligence — using MISP and STIX/TAXII to manage, share, and consume intel with the wider community.
difficulty: intermediate
tags: [threat-intel, misp, sharing, stix, taxii]
tools: [misp, stix, taxii]
---

## Purpose

Threat intelligence is more valuable shared — an indicator one organisation sees today warns the community tomorrow. But sharing needs structure: a platform to manage intel, standard formats so tools interoperate, and trust/handling rules so sensitive intel isn't over-shared. This skill covers running a threat-intel platform (MISP is the open standard) and exchanging intelligence via STIX/TAXII, the plumbing that turns a collection of indicators into a collaborative defence.

## When to use it

Standing up a threat-intel capability, joining a sharing community (ISAC, sector group), or integrating intel feeds into your tools. It's the operational backbone that makes intel manageable and interoperable, connecting collection (IoC vetting) to consumption (detection, enrichment).

## Procedure

1. **Use a platform to manage intel — don't do it in spreadsheets.** A Threat Intelligence Platform (MISP is the widely-used open-source standard) stores indicators and events, tracks their context and relationships, handles sharing, and integrates with your tools. It's the central store the whole programme organises around.
2. **Structure intel as events with context.** In MISP, related indicators group into events (an incident, a campaign, an actor's activity) with attributes, context, and relationships — not a flat indicator list. This structure is what makes intel usable rather than a pile of IoCs.
3. **Use standard formats for interoperability.** STIX (the structured intel format) and TAXII (the transport protocol) let platforms and tools exchange intel automatically. Producing and consuming STIX/TAXII means your intel flows between organisations and into your detection/blocking tools without manual reformatting.
4. **Join sharing communities and set trust levels.** ISACs, sector groups, and MISP sharing communities exchange intel among trusted members. Configure sharing/distribution levels carefully — some intel is shareable widely, some only within a trusted group, some not at all (sensitive/attributable). Handling designations (like TLP — Traffic Light Protocol) govern who can see what.
5. **Respect handling and sensitivity — the sharing discipline.** Over-sharing sensitive intel (an ongoing investigation, attributable data, a source's information) can burn sources, tip off adversaries, or breach agreements. Honour TLP markings on intel you receive, and mark what you share appropriately. Sharing is valuable but must respect trust boundaries.
6. **Automate consumption into defence.** Intel in the platform should flow automatically into detection and blocking (feeds the enrichment and detection-mapping skills) — vetted indicators become alerts and blocks, actor TTPs become detection priorities. Intel that stays in the platform helps no one.
7. **Contribute back.** Sharing is reciprocal; contributing your vetted indicators and observations (respecting handling rules) strengthens the community and your standing in it.

## Cheatsheet

```
intel is more valuable SHARED (your indicator today = community's warning tomorrow)
  needs: platform + standard formats + trust/handling rules

PLATFORM: MISP (open-source standard TIP) — store, context, relationships, share, integrate
  (not spreadsheets)
  structure as EVENTS with context (campaign/actor/incident), not a flat IoC list

STANDARDS: STIX (format) + TAXII (transport) -> automatic exchange between orgs + into tools

SHARING COMMUNITIES: ISACs, sector groups, MISP communities (trusted members)
  set distribution/trust levels carefully

HANDLING DISCIPLINE (the key rule): TLP (Traffic Light Protocol) — who can see what
  over-sharing sensitive intel = burn sources / tip off adversary / breach agreement
  honour TLP on received intel ; mark what you share

automate CONSUMPTION into detection+blocking (intel in the platform helps no one)
CONTRIBUTE back (reciprocal ; respect handling rules)
```

## Reading the practice

- **Intel managed in a real platform (MISP) with structured events and context** = usable, shareable, integrable intelligence; the alternative (spreadsheets of indicators) doesn't scale, lacks context, and can't share automatically.
- **STIX/TAXII in use** = intel flows automatically between organisations and into your tools; without standard formats, exchange is manual reformatting that doesn't scale.
- **Over-shared sensitive intel** = a serious mistake — sharing attributable data, an ongoing investigation, or a source's information can burn sources, tip off the adversary, or breach trust agreements. Honour TLP and handling markings; this discipline is what keeps sharing communities functioning.
- **Received intel ignoring TLP markings** = a trust violation that gets you excluded from communities; respect the handling designations on intel you consume.
- **Intel sitting in the platform, not flowing to defence** = collection without value; the point is automated consumption into detection and blocking.
- **A structured platform, standard formats, respected handling, automated consumption, and reciprocal contribution** = a functioning intel-sharing capability.

## Pitfalls

- **Managing intel in spreadsheets.** It doesn't scale, lacks context and relationships, and can't share or integrate automatically. Use a platform (MISP).
- **Ignoring standard formats.** Without STIX/TAXII, intel exchange is manual and doesn't flow into tools; adopt the standards for interoperability.
- **Over-sharing sensitive intel.** Sharing attributable, investigative, or source-sensitive intel breaches trust, burns sources, and can tip off adversaries. Respect TLP and handling rules — this is the core sharing discipline.
- **Violating received TLP markings.** Mishandling intel others shared gets you excluded from communities; honour the designations.
- **Collecting/sharing without consuming.** Intel that doesn't flow into detection and blocking is inert; automate consumption into defence.
- **Not contributing back.** Sharing is reciprocal; consuming without contributing weakens the community and your position in it.

## References

- MISP documentation (misp-project.org) — events, sharing, feeds, warninglists
- STIX/TAXII specifications (OASIS) and the Traffic Light Protocol (TLP) standard
- The ioc-collection-and-vetting, enrichment-pipelines, and mapping-intel-to-detection skills
- FIRST and sector ISAC sharing guidance
