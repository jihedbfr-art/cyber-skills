---
format: "v2"
name: "reporting-culture"
title: "Reporting Culture"
title_fr: "Culture du signalement"
description: "Use when building a culture where people report suspicious activity — making reporting easy, safe, and rewarded, so users become a detection layer instead of hiding mistakes."
description_fr: "À utiliser pour bâtir une culture où chacun signale les activités suspectes — en rendant le signalement facile, sans risque et valorisé, afin que les utilisateurs deviennent une couche de détection au lieu de cacher leurs erreurs."
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

The most valuable anti-phishing control isn't technical — it's a workforce that reports suspicious emails, calls, and messages, turning every employee into a sensor. But people only report if it's easy, if they're not punished for their own mistakes, and if reporting visibly matters. This skill covers building that reporting culture, the human detection layer that catches what technical controls miss — and it depends far more on how people are treated than on any tool.

### When to use it

Building the human layer of security defence, underpinning the whole social-engineering-defence domain. A strong reporting culture is what makes phishing simulations, awareness, and human vigilance actually pay off — without it, users see threats and stay silent.

### Procedure

1. **Make reporting trivially easy — remove friction.** A one-click report button in the email client (report phishing) is the standard; if reporting requires forwarding to an address, filling a form, or figuring out who to tell, most people won't bother. The easier it is, the more reports you get. Friction is the enemy of reporting.
2. **Make it safe — never punish the reporter, even for their own mistake.** Someone who clicked a phishing link and *then* reports it is doing exactly the right thing — reporting is what limits the damage. If reporting a mistake leads to punishment or shame, people hide mistakes, and you lose the early warning. Psychological safety is the foundation: a reported click is a gift, not an offence.
3. **Reward and recognise reporting.** Celebrate good catches (a user who spotted a real phish), thank every reporter, and make reporting feel valued. Positive reinforcement builds the behaviour; a report that vanishes into a void or gets a cold auto-reply discourages the next one.
4. **Close the loop — show reporting matters.** Tell reporters what happened ("thanks, that was a real phishing campaign, we've blocked it and warned others"). When people see their report led to action, they report again. Visible impact is what sustains the culture.
5. **Treat reports as a detection source.** User reports are genuine security telemetry — often the earliest signal of a phishing campaign (users report it before filters catch it). Wire reports into the SOC/triage (the SOC domain) so they're acted on, not ignored. This also makes closing the loop natural.
6. **Frame everyone as part of the defence, not a weak link.** The narrative shapes behaviour: "you're our sensors, and reporting is how you protect us" builds engagement, while "users are the weakest link" breeds disengagement and blame. People rise to being trusted defenders.
7. **Reinforce continuously** through awareness and non-punitive simulations (the phishing-simulation skill), always measuring and celebrating reporting rate as the win.

### Cheatsheet

```
most valuable anti-phishing control = a workforce that REPORTS (every employee = a sensor)
  catches what technical controls miss. depends on how people are TREATED, not tooling.

people report only if: EASY + SAFE + it visibly MATTERS

build it
  EASY: one-click report button (friction = the enemy ; forms/figuring-out-who = nobody bothers)
  SAFE: NEVER punish the reporter — even for their own mistake
    (clicked THEN reported = exactly right ; punishment -> people HIDE mistakes -> lose early warning)
    a reported click is a GIFT, not an offence. psychological safety = the foundation.
  REWARD: celebrate catches, thank every reporter, make it feel valued
  CLOSE THE LOOP: tell them what happened ("real campaign, blocked, warned others") -> they report again
  DETECTION SOURCE: reports = real telemetry (often EARLIEST signal, before filters) -> wire into SOC
  FRAME: "you're our sensors" (engagement) NOT "users are the weakest link" (blame + disengagement)
  reinforce continuously ; measure + celebrate REPORTING RATE as the win
```

### Reading the culture

- **A one-click report button, widely used** = friction removed, reports flowing; the single most impactful enabler. If reporting requires forwarding or a form, report rates stay low regardless of awareness.
- **Users who click and then report** = exactly the behaviour you want; reporting limits the damage. If the culture punishes this, people hide mistakes and you lose the early warning — the foundational failure.
- **Reports vanishing into a void / cold auto-replies** = discourages future reporting; closing the loop (telling reporters what happened) is what sustains the behaviour. Visible impact matters.
- **A "users are the weakest link" narrative** = breeds disengagement and blame; reframing people as sensors and defenders builds the engagement that produces reports. Framing shapes behaviour.
- **User reports being the earliest signal of a campaign** = the payoff; a strong reporting culture catches phishing before filters do, wired into the SOC as real telemetry.
- **Easy, safe, rewarded, loop-closed reporting measured by reporting rate** = the human detection layer working — the state to build.

### Pitfalls

- **Punishing the reporter.** The foundational error — punishing someone who reports their own mistake teaches people to hide mistakes, losing the early warning that limits damage. A reported click is a gift; never punish it.
- **Friction in reporting.** If it's not one-click, most people won't report; forms and forwarding kill report rates. Remove friction.
- **Not closing the loop.** Reports that vanish into a void discourage the next one; tell reporters what happened so they see it matters. Visible impact sustains the culture.
- **The "weakest link" narrative.** It breeds blame and disengagement; frame people as sensors and defenders to build engagement and reports.
- **Ignoring reports as telemetry.** User reports are often the earliest signal of a campaign; not wiring them into the SOC wastes real detection value.
- **Optimising for not-clicking over reporting.** Reporting is the win (a clicker who reports is valuable); measure and celebrate reporting rate.

### References

- The phishing-simulation-programme, phishing-email-analysis, and SOC alert-triage skills
- Research on psychological safety and security-awareness effectiveness
- NIST and SANS security-awareness / human-factor guidance
- The vishing-and-smishing-awareness skill (reporting across all channels)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.