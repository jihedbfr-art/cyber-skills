---
name: operationalising-a-hunt
domain: 20-threat-hunting
description: Use when turning a hunt into lasting value — converting findings into detections, documenting the hunt, and feeding results back so the programme improves instead of repeating itself.
difficulty: intermediate
tags: [threat-hunting, operationalising, detection, documentation, maturity]
tools: []
---

## Purpose

A hunt that finds something and stops there is a wasted opportunity — you'll hunt the same thing again next month. The point of a mature hunting programme is that each hunt makes the *next* attack easier to catch: findings become automated detections, methods become repeatable, and results shrink the space of the unknown. This skill covers operationalising hunts — the maturity practice that turns hunting from a series of one-off searches into a compounding capability. It's the closing discipline of the domain.

## When to use it

At the end of every hunt, whatever the outcome, and as the organising principle of a hunting programme. It's what separates ad-hoc hunting (find something, move on) from a programme that steadily improves detection coverage and visibility.

## Procedure

1. **Turn repeatable findings into detections — the core move.** If a hunt found something by a method that could run automatically, write a detection for it (the detection-engineering domain). You should never have to *hunt* for the same thing twice — once you can describe it, automate it. This is how hunting continuously feeds and improves automated detection.
2. **Document every hunt — including the ones that found nothing.** Record the hypothesis, the data and method, and the outcome (threat found / clean / visibility gap). A "clean" hunt with good data is a real assurance result worth recording, and documentation makes hunts repeatable and prevents re-hunting the same ground blindly.
3. **Feed visibility gaps back as telemetry requests.** When a hunt couldn't conclude because the data wasn't there, that gap is a finding — route it to logging/tooling improvements (the log-source-coverage skill) so future hunts (and detections) can see what this one couldn't.
4. **Feed threat findings into IR and remediation.** A hunt that finds a real threat becomes an incident; hand off cleanly (the IR skills), and the intrusion's IoCs and techniques enrich threat intel and detection.
5. **Track coverage and build on prior hunts.** Record what's been hunted (against ATT&CK — the hunting-with-attack skill) so hunts accumulate into coverage rather than repeating, and each new hunt targets uncovered ground.
6. **Refine the hunting process itself.** What worked, what data was missing, what method was efficient — feed lessons back so the programme's *technique* improves, not just its coverage. Mature hunting gets better at hunting.
7. **Measure the programme's value.** Detections created, visibility gaps closed, threats found, and coverage gained are the outputs that justify hunting and show it's compounding (not just activity). A hunt programme that produces no detections or coverage growth isn't maturing.

## Cheatsheet

```
a hunt that finds something and STOPS = wasted (you'll hunt it again next month)
  mature hunting: each hunt makes the NEXT attack easier to catch (compounding)

operationalise every hunt (whatever the outcome)
  1. repeatable finding -> DETECTION (never hunt the same thing twice)  <- core move
  2. DOCUMENT every hunt (incl. found-nothing): hypothesis + data/method + outcome
       clean hunt w/ good data = assurance result ; prevents re-hunting blindly
  3. visibility gap -> TELEMETRY request (log-source-coverage)
  4. threat found -> IR handoff + IoCs/techniques -> intel + detection
  5. track COVERAGE (ATT&CK) -> hunts accumulate, target uncovered ground
  6. refine the PROCESS (what worked / data missing / efficient method)
  7. MEASURE value: detections created, gaps closed, threats found, coverage gained
       (no detections/coverage growth = not maturing, just activity)

signature of maturity: hunting continuously FEEDS automated detection.
```

## Reading the practice

- **A hunt that finds something and produces no detection** = the value leaked away; you'll hunt the same thing again. The defining maturity move is turning repeatable findings into automated detections — once you can describe it, you shouldn't have to hunt it. This is the biggest missed opportunity in immature programmes.
- **Hunts that aren't documented** = they don't compound; you re-hunt the same ground, can't show coverage, and lose the "clean with good data" assurance results. Document every hunt, including empty ones.
- **A "found nothing" hunt treated as a failure** = a misread; a clean hunt with solid telemetry is genuine assurance ("we hunted X, had the data, found nothing"). Record it as a positive result.
- **Visibility gaps not fed back** = the same blind spot blocks the next hunt; routing gaps to telemetry improvements is how hunting expands what the whole SOC can see.
- **No coverage tracking** = hunts repeat and can't show progress; tracking against ATT&CK makes them cumulative and strategic.
- **A programme producing detections, closing gaps, and growing coverage** = mature, compounding hunting — the outputs prove it's improving the defence, not just performing activity.

## Pitfalls

- **Hunting without operationalising.** Finding something and moving on wastes it — you'll hunt it again. Turn repeatable findings into detections; that's the whole point of maturity.
- **Not documenting hunts.** Undocumented hunts don't compound: you re-hunt ground, lose assurance results, and can't demonstrate coverage. Document every one.
- **Treating empty hunts as failures.** A clean hunt with good telemetry is real assurance; not recording it discards value and invites re-hunting.
- **Ignoring visibility gaps.** A hunt that couldn't conclude reveals a blind spot; not routing it to telemetry improvements leaves it blocking future hunts and detections.
- **No coverage tracking or metrics.** Without them, hunts repeat and you can't show the programme is compounding. Track coverage and measure outputs (detections, gaps closed, threats found).

## References

- The detection-engineering domain (turning findings into detections) and its mapping-to-attack skill
- The hunting-with-attack, hypothesis-driven-hunting, and log-source-coverage skills
- The incident-response domain (threat-found handoff) and threat-intelligence domain
- SANS threat hunting maturity and MITRE hunting resources
