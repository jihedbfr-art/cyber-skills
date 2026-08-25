---
format: "v2"
name: "chain-of-custody"
title: "Chain Of Custody"
title_fr: "Chaîne de possession"
description: "Use when handling digital evidence that might be challenged — documenting who had it, when, and why, so the evidence and your findings hold up legally and procedurally."
description_fr: "À utiliser pour manipuler une preuve numérique susceptible d'être contestée — documenter qui l'a détenue, quand, et pourquoi, pour que la preuve et vos conclusions tiennent juridiquement et procéduralement."
domain: "23-digital-forensics"
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

Brilliant forensic analysis is worthless if the evidence gets thrown out. Chain of custody is the documented, unbroken record of who handled a piece of evidence, when, why, and how it was protected — the process that lets a court, regulator, or tribunal trust that the evidence is authentic and unaltered. This skill covers maintaining chain of custody so findings survive challenge. It's the discipline that wraps every other forensics skill; without it, the technical work may not count.

### When to use it

Any investigation where findings might be challenged — legal proceedings, HR/disciplinary cases, regulatory matters, insurance claims. And since you often don't know at the outset whether a case will go legal, the safe default is to maintain chain of custody from the first moment you touch evidence. Treating it as if it *will* be challenged is the prudent posture.

### What chain of custody must establish

For every piece of evidence, an unbroken record answering:

- **What** it is — precise identification (device make/model/serial, image hash, unique evidence number).
- **Who** collected it, and every person who has handled it since.
- **When** each transfer/handling occurred (date and time).
- **Where** it was collected from and where it's been stored.
- **Why** each handling occurred (collected, analysed, transferred).
- **How** its integrity was protected — hashes, write-blockers, secure storage, tamper-evident packaging.

Any unexplained gap in this record is what opposing counsel attacks.

### Procedure

1. **Document from the moment of collection.** Record the evidence details, the collector, the time, and the source at acquisition — the first custody entry. Photograph the device and its condition where relevant.
2. **Establish integrity immediately** — hash the evidence (the disk-imaging skill) and use write-blockers so you can prove it wasn't altered. The hash is the mathematical anchor of the whole chain: it proves the evidence today is the evidence collected.
3. **Log every handling and transfer.** Each time evidence changes hands or is accessed, record who, when, and why, with both parties' acknowledgement. An unbroken sequence of signed transfers is the chain.
4. **Work on copies, preserve the original.** Keep the master evidence sealed and untouched; analyse verified working copies. Re-hash to prove the copy matches. This keeps the original defensible while you work.
5. **Store securely** — access-controlled, tamper-evident storage, so the record of "who could have touched it" is limited and provable. Physical evidence in sealed bags; digital evidence in controlled repositories.
6. **Maintain it for the whole lifecycle** — through analysis, reporting, any legal process, and eventual disposal. The chain doesn't end when analysis does.
7. **Keep the documentation itself defensible** — accurate, contemporaneous (recorded at the time, not reconstructed later), and complete. A custody log written from memory afterwards is weak.

### Cheatsheet

```
default posture: you often don't know if a case goes legal -> maintain chain of
                 custody from FIRST contact with evidence. Treat it as if it WILL
                 be challenged.

every evidence item must record (unbroken):
  WHAT   precise ID: make/model/serial, image HASH, evidence number
  WHO    collector + every handler since
  WHEN   date+time of each handling/transfer
  WHERE  collected from + stored where
  WHY    reason for each handling (collect/analyse/transfer)
  HOW    integrity: HASH + write-blocker + secure/tamper-evident storage

practices
  hash immediately (mathematical anchor: proves unaltered)
  work on COPIES, preserve+seal the original ; re-hash to prove match
  log every transfer with both parties' acknowledgement
  contemporaneous documentation (recorded at the time, not from memory)

any UNEXPLAINED GAP = what gets the evidence thrown out.
```

### Reading the custody record

- **An unbroken, hashed, contemporaneous record** = evidence that holds up; the technical findings rest on a defensible foundation. This is the goal — the boring paperwork that makes the analysis count.
- **A gap in the chain** (an unexplained handoff, a period nobody accounted for, a missing transfer log) = exactly what opposing counsel targets; a single gap can invalidate otherwise-perfect evidence. The chain is only as strong as its weakest link.
- **No hash at collection** = you can't prove the evidence is unaltered; the integrity anchor is missing and the whole chain is weakened.
- **Analysis performed on the original** = the evidence may have been changed by the analysis itself; work on copies and preserve the master, or defensibility collapses.
- **A custody log reconstructed from memory** = weak and attackable; contemporaneous records (made at the time) are what carry weight.
- **Evidence in uncontrolled storage** = anyone could have accessed/altered it; without access control the "who could have touched it" question has no good answer.

### Pitfalls

- **Deciding chain of custody isn't needed "because it's not legal".** You frequently can't know that at the start, and a case can turn legal later — at which point unmaintained custody can't be retrofitted. Maintain it by default from first contact.
- **Any gap in the record.** Unexplained handling periods or missing transfer logs are the classic way evidence gets excluded. Log every touch.
- **No integrity hash.** Without it you can't prove the evidence is unaltered; hash at collection and re-verify.
- **Working on the original.** It risks altering the evidence; analyse copies, keep the master sealed and verified.
- **Non-contemporaneous documentation.** A custody log written afterwards from memory is weak; record at the time.
- **Uncontrolled storage.** Evidence anyone could access undermines the chain; use access-controlled, tamper-evident storage.

### References

- NIST SP 800-86 (forensic techniques) and SWGDE best practices
- ISO/IEC 27037 (handling of digital evidence)
- The disk-imaging-and-hashing and evidence-preservation skills
- ACPO / jurisdiction-specific digital evidence guidelines

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.