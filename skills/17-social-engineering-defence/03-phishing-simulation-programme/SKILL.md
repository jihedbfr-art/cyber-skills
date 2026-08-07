---
name: phishing-simulation-programme
domain: 17-social-engineering-defence
description: Use when running authorised phishing simulations — testing and training users in a way that builds resilience and a reporting culture, not fear and resentment.
difficulty: intermediate
tags: [social-engineering, phishing, simulation, awareness, training]
tools: [gophish]
---

## Purpose

Phishing simulations send benign fake-phishing emails to your own users to measure susceptibility and train recognition. Done well, they build a workforce that spots and reports phishing; done badly — as "gotcha" tests that punish and humiliate — they breed fear, resentment, and a culture where people hide mistakes. This skill covers running a phishing simulation programme that actually improves resilience, which depends far more on the *culture* than the tooling.

## When to use it

Building the human layer of phishing defence, alongside the technical controls (SPF/DKIM/DMARC, detonation). Phishing is a top attack vector, so training people to recognise and report it is high-value — but only if the programme builds trust rather than destroying it.

## Procedure

1. **Define the goal as building resilience, not catching people out.** The purpose is a workforce that recognises and *reports* phishing — not a click-rate leaderboard or a way to punish. This framing shapes every other decision; a "gotcha" programme that humiliates people damages the reporting culture you most need.
2. **Get authorisation and set it up properly.** Simulations send deceptive emails to employees; ensure it's authorised, and use a platform (GoPhish or commercial) to send, track, and manage safely.
3. **Design realistic but fair scenarios.** Base simulations on real phishing techniques (and the pretexts a real attacker could build from your public footprint — the social-media-profiling skill), at a difficulty that teaches. Avoid cruel lures (fake bonuses, fake layoffs, fake emergencies) that cause real distress and destroy trust — the backlash outweighs any training value.
4. **Make the teachable moment supportive, not punitive.** When someone clicks, the response should be immediate, brief, supportive training ("here's what to look for next time") — not public shaming, punishment, or a mark on their record. People who fear punishment hide mistakes; people who are supported report them. This is the crux.
5. **Reward reporting, and make it easy.** The real goal is reporting, not just not-clicking. Celebrate and make it trivial to report suspected phishing (a one-click report button — the reporting-culture skill). A user who clicks but reports is more valuable than one who neither clicks nor reports.
6. **Measure the right things.** Track *reporting rate* (rising is the win) alongside click rate, and trends over time, not individual shaming. Improvement in recognition and reporting is the metric, not a click-rate scoreboard.
7. **Run continuously and vary.** One-off simulations don't build lasting resilience; ongoing, varied simulations keep recognition sharp — but at a cadence that trains without fatiguing or feeling like harassment.

## Cheatsheet

```
simulations = benign fake-phishing to your users -> measure susceptibility + TRAIN recognition
  done well: workforce that spots + REPORTS phishing
  done badly (gotcha/punish/humiliate): fear, resentment, people HIDE mistakes
  -> success depends on CULTURE far more than tooling

run it
  GOAL = build RESILIENCE + reporting, not catch people out (framing shapes everything)
  authorised + a platform (GoPhish/commercial)
  realistic but FAIR scenarios (real techniques ; AVOID cruel lures — fake bonus/layoff/emergency
    cause real distress + destroy trust ; backlash > training value)
  teachable moment = SUPPORTIVE not punitive (immediate brief training, no shaming/punishment)
    fear -> hide mistakes ; support -> report. THE CRUX.
  REWARD reporting + make it EASY (one-click report) ; clicker-who-reports > neither
  MEASURE: reporting RATE (rising = win) + click rate + trends — NOT individual shaming
  CONTINUOUS + varied (one-off = no lasting resilience) at a non-harassing cadence
```

## Reading the programme

- **A "gotcha" programme that punishes and shames clickers** = the core failure; it breeds fear and resentment, and people start hiding mistakes and distrusting security — destroying the reporting culture that's the actual goal. Supportive, not punitive, is the crux.
- **Cruel lures** (fake bonuses, fake layoffs, fake emergencies) = cause real employee distress and a backlash that outweighs any training value; the programme becomes hated and counterproductive. Design fair scenarios.
- **Measuring only click rate, individually** = the wrong metric and a shaming tool; track *reporting rate* (rising is the real win) and trends, not a scoreboard of who clicked.
- **A rising reporting rate over time** = the programme working; more people recognising and reporting phishing is the goal, more than a falling click rate alone.
- **Reporting hard or unrewarded** = you're optimising for not-clicking instead of reporting; a user who clicks but reports is valuable. Make reporting easy and celebrated.
- **Supportive teachable moments, rewarded reporting, reporting-rate metrics, continuous fair simulations** = a programme that builds resilience and trust — the state to aim for.

## Pitfalls

- **Punitive "gotcha" programmes.** Punishing and shaming clickers breeds fear, resentment, and hidden mistakes, destroying the reporting culture you need. Make the teachable moment supportive.
- **Cruel lures.** Fake bonuses/layoffs/emergencies cause real distress and backlash that outweighs training value; the programme becomes counterproductive. Design fair scenarios.
- **Measuring/optimising for click rate alone.** It ignores reporting (the real goal) and becomes a shaming metric. Track reporting rate and trends.
- **Not rewarding or easing reporting.** Optimising for not-clicking misses that reporting is the win; a clicker who reports is valuable. Make reporting trivial and celebrated.
- **One-off simulations.** They don't build lasting resilience; run continuously and varied — but not so aggressively it feels like harassment.
- **Treating tooling as the programme.** Success is about culture (supportive, reporting-focused) far more than the platform. Get the culture right.

## References

- The reporting-culture, phishing-email-analysis, and social-media-profiling skills
- GoPhish and commercial phishing-simulation platform documentation
- Research on effective (non-punitive) security awareness training
- SANS and NIST security-awareness guidance
