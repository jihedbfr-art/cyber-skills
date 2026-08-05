---
name: timeline-analysis
domain: 23-digital-forensics
description: Use when reconstructing the sequence of an incident — building a super-timeline from all artefacts so events across sources line up chronologically and the story emerges.
difficulty: intermediate
tags: [forensics, timeline, plaso, super-timeline, reconstruction]
tools: [plaso, log2timeline, timesketch]
---

## Purpose

An investigation ultimately has to answer "what happened, in what order?" — and the evidence for that is scattered across dozens of artefacts (logs, filesystem, registry, browser, memory) each with its own timestamps. Timeline analysis pulls them into one chronological view so the sequence becomes visible: the phish arrived, the macro ran, the process spawned, the connection went out, persistence was set. This skill covers building and reading a super-timeline, the technique that turns scattered artefacts into a story.

## When to use it

Once you have artefacts from disk/memory analysis and need to establish sequence and causality — which is most non-trivial investigations. It's the synthesis step that sits on top of the Windows/Linux artefact skills and produces the narrative IR and reports need.

## Procedure

1. **Collect timestamped data from all sources.** The power of a *super*-timeline is breadth — filesystem timestamps ($MFT), event logs, registry, browser history, shell history, and more, unified. `log2timeline`/plaso parses many artefact types into one timeline:
   ```
   log2timeline.py timeline.plaso image.raw       # ingest artefacts -> plaso storage
   psort.py -o l2tcsv timeline.plaso > timeline.csv   # render to a reviewable timeline
   ```
2. **Narrow to the relevant window.** A full super-timeline is enormous (millions of events); filter to the incident timeframe and the systems/users involved so the signal is findable. Analysis tools (Timesketch) let you filter, search, and pivot without drowning.
3. **Anchor on a known event.** Start from something you're sure about — the alert time, a known-malicious file's creation, a suspicious logon — and expand outward in both directions to see what led to it and what followed.
4. **Read the sequence for the story.** Look for the causal chain: an email/download, then execution, then a spawned process, then a network connection, then persistence. Events that cluster tightly in time around the anchor are usually related.
5. **Corroborate across artefact types.** A single timestamp can be wrong or faked; when filesystem, log, and registry artefacts agree on a sequence, confidence is high. Divergence (a file "modified" before it was "created") flags timestomping.
6. **Mind timestamp pitfalls** — time zones (normalise to UTC), clock skew between systems, and the different timestamp types (creation vs modification vs access vs metadata-change). Getting these wrong corrupts the sequence.
7. **Produce the narrative** — the ordered sequence of events with times becomes the backbone of the investigation report and the IR scoping.

## Cheatsheet

```
goal: scattered artefacts -> ONE chronological view -> the story/sequence

build the super-timeline (breadth is the power)
  log2timeline.py out.plaso image.raw     ingest filesystem+logs+registry+browser+...
  psort.py -o l2tcsv out.plaso > tl.csv   render reviewable timeline
  Timesketch                               filter / search / pivot / collaborate

analyse
  1. narrow to incident window + relevant systems/users (full TL = millions of events)
  2. anchor on a KNOWN event (alert time, malicious file, suspicious logon)
  3. expand outward -> read the causal chain
       email/download -> execution -> spawned process -> network -> persistence
  4. corroborate across artefact TYPES (agreement = confidence; divergence = timestomp)

pitfalls to control: TIME ZONES (normalise UTC) | clock skew | timestamp TYPES
  ($SI vs $FN, create vs modify vs access vs ctime)
```

## Reading the timeline

- **A tight cluster of events around the anchor** = usually the incident unfolding; events close in time to a known-malicious action are typically related. This clustering is how the story surfaces.
- **A clean causal chain** (download → execute → spawn → connect → persist) = the reconstructed attack sequence — the deliverable timeline analysis exists to produce.
- **Timestamp divergence** (a file modified before created, $STANDARD_INFORMATION vs $FILE_NAME mismatch) = timestomping/anti-forensics; the inconsistency is itself evidence and tells you not to trust that artefact's times alone.
- **A gap where you'd expect activity** = possible log clearing or evidence destruction; absence in the timeline is meaningful.
- **Events that don't line up across time zones** = a normalisation error (or genuine clock skew); fix the time handling before trusting the sequence, or you'll misread causality.
- **Corroborating artefacts agreeing on order** = high-confidence sequence; the multi-source agreement is what makes the reconstruction defensible.

## Pitfalls

- **Time-zone and skew errors.** The most damaging timeline mistake — mixed time zones or clock skew scrambles the order and can reverse cause and effect. Normalise to UTC and account for skew.
- **Drowning in the full super-timeline.** Millions of events are unusable raw; narrow to the window, systems, and users, and anchor on a known event.
- **Trusting a single timestamp.** Times can be wrong or faked; corroborate across artefact types, and treat divergence as a timestomping signal.
- **Confusing timestamp types.** Creation vs modification vs access vs metadata-change mean different things; misreading which is which corrupts the sequence.
- **Building a timeline without a question/anchor.** Aimlessly scrolling a timeline finds nothing; anchor on something known and expand.

## References

- Plaso / log2timeline and Timesketch documentation
- SANS timeline analysis resources (FOR508)
- The windows-artefacts, linux-artefacts, memory-forensics, and anti-forensics-awareness skills
- MITRE ATT&CK — T1070.006 (Timestomp)
