---
name: data-stacking
domain: 20-threat-hunting
description: Use when hunting with frequency analysis — stacking data to surface the rare outlier across a large dataset, one of the most reliable techniques for finding what doesn't belong.
difficulty: intermediate
tags: [threat-hunting, stacking, frequency-analysis, outliers, technique]
tools: []
---

## Purpose

One of the most dependable hunting techniques is deceptively simple: count how often each value appears across a large dataset, and look at the rare ones. Attackers, by nature, do things that are uncommon in your environment — a tool nobody else runs, a service name that appears once, a scheduled task on a single host. Data stacking (frequency analysis, "long tail analysis") surfaces those rare outliers by counting. This skill covers the technique that turns "find the needle" into "sort by frequency and look at the bottom."

## When to use it

Whenever you're hunting across many similar entities (hundreds of hosts, thousands of processes) and the malicious thing is likely to be *rare*. It's a workhorse technique underneath endpoint hunting, persistence hunting, and more — simple, powerful, and applicable to almost any dataset with a lot of instances.

## Procedure

1. **Pick a dataset with many instances and a field where rare = suspicious.** Stacking works when normal produces high-frequency values and malicious produces rare ones: process names across a fleet, service names, scheduled task names, DLLs loaded, autorun entries, parent-child process pairs, user-agents. The more homogeneous the environment, the sharper the technique.
2. **Count the frequency of each value.** Aggregate and count — how many hosts run each process, how often each service name appears, how many times each autorun entry is seen. This is a simple group-by-and-count, but the framing is what makes it powerful.
3. **Examine the long tail — the rare values.** Sort by frequency and look at the bottom: the values seen once or a handful of times. In a fleet of identical corporate machines, a process on *one* host that's on no others is exactly the kind of outlier attackers create. The rarity is the signal.
4. **Understand why rarity works here.** Legitimate enterprise software is deployed broadly (high frequency); attacker tools, custom malware, and one-off persistence are, by nature, rare (low frequency). Homogeneity amplifies this — the more your endpoints *should* look alike, the more a rare thing stands out.
5. **Investigate the outliers.** A rare value is a *candidate*, not a conclusion — some rare things are benign (a legitimate tool on one admin's machine, a niche business app). Investigate each outlier to determine whether it's malicious.
6. **Combine with other signals.** Stacking surfaces the rare; combine with context (what is this process, where's it running, who ran it) and other hunt techniques to confirm. Rarity plus a suspicious characteristic is a strong lead.
7. **Operationalise findings** — a malicious outlier becomes an IoC/detection; and the stacking itself can be automated as a recurring hunt (surface new rare values periodically).

## Cheatsheet

```
simplest reliable hunt: COUNT each value across a big dataset -> look at the RARE ones
  attackers do UNCOMMON things -> the outlier is often the attacker
  ("long tail" / frequency analysis)

1. dataset with many instances + field where RARE = suspicious
     process names | service names | scheduled tasks | loaded DLLs | autoruns
     | parent-child pairs | user-agents
2. COUNT frequency (group-by + count)
3. examine the LONG TAIL (seen once / a handful) — process on 1 host of 1000s = outlier
4. why it works: legit software = broadly deployed (frequent) ;
     attacker tools/malware/one-off persistence = RARE. homogeneity amplifies.
5. outlier = CANDIDATE not conclusion (some rare things benign) -> investigate
6. COMBINE with context (what/where/who) + other techniques -> confirm
7. operationalise: malicious outlier -> IoC/detection ; automate recurring stack
```

## Reading the stack

- **A value in the long tail** (a process/service/task seen on one or a few hosts out of many) = the outlier the technique exists to surface; in a homogeneous environment, the rare thing is disproportionately likely to be the attacker. Investigate the bottom of the frequency list.
- **A rare process on a single host that's on no others** = a classic stacking find; attacker tools and custom malware are, by definition, not broadly deployed, so they sit in the tail.
- **High-frequency values** = the normal, broadly-deployed legitimate software; these are the noise, and the point of stacking is to look *past* them at the rare.
- **A rare-but-benign outlier** (a legitimate niche tool, an admin's one-off utility) = expected; rarity produces candidates, not verdicts. Investigate to separate benign rare from malicious rare.
- **A homogeneous environment** (identical corporate builds) = where stacking is sharpest; the more things *should* look alike, the more a rare value stands out. Heterogeneous environments have a noisier tail.
- **Rarity plus a suspicious characteristic** (rare process + unusual path + odd parent) = a strong lead; combining stacking with context confirms the outlier.

## Pitfalls

- **Treating rare as malicious.** Rarity surfaces candidates, not conclusions; plenty of rare things are benign (niche tools, one-off legitimate activity). Investigate each outlier rather than assuming.
- **Stacking a heterogeneous dataset.** The technique is sharpest when normal is homogeneous; in a highly varied environment the long tail is large and noisy, diluting the signal. Segment to comparable groups.
- **Stacking the wrong field.** It only works where rare = suspicious; a field where rarity is normal (unique IDs, timestamps) produces meaningless results. Pick fields where attackers create rarity.
- **Ignoring context.** A rare value alone is weak; combine with what/where/who to confirm. Rarity plus a suspicious characteristic is the strong lead.
- **Not automating.** Stacking is easy to repeat; automating it as a recurring hunt (surface new rare values) catches new outliers over time rather than a one-off look.

## References

- SANS threat hunting resources (frequency analysis / long-tail analysis)
- The endpoint-hunting, anomaly-baselining, and operationalising-a-hunt skills
- The detection reducing-false-positives skill (rarity and baselining overlap)
- MITRE ATT&CK (the techniques whose artifacts stack well: persistence, execution)
