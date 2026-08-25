---
format: "v2"
name: "disk-imaging-and-hashing"
title: "Disk Imaging And Hashing"
title_fr: "Imagerie disque et hachage"
description: "Use when you need a forensically sound copy of a disk before analysis — acquiring a bit-for-bit image, hashing to prove integrity, and preserving chain of custody."
description_fr: "À utiliser quand il faut une copie légalement recevable d'un disque avant analyse — acquérir une image bit à bit, la hacher pour prouver son intégrité, et préserver la chaîne de possession."
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

Forensics stands or falls on acquisition. If the image isn't a faithful, verifiable copy — or if you worked on the original and changed it — nothing you find afterward holds up. This skill covers taking a bit-for-bit image, hashing it to prove it's unaltered, and handling it so the evidence stays defensible. Everything in the forensics domain assumes you did this first, correctly.

### When to use it

Whenever a disk (or other storage) is evidence: an incident where the host will be analysed, a legal matter, any case where findings might be challenged. Acquire before you analyse — you work on copies, never the original.

### Procedure

1. **Preserve the original.** Use a **write blocker** (hardware or a verified software one) between the evidence drive and your workstation so the acquisition process can't modify the source. Document the device, serial number, and condition before you start.
2. **Record identifying details** — make, model, serial, capacity, and the case/evidence number — as the first chain-of-custody entry. Photograph the drive and its connections if this may go to court.
3. **Acquire a bit-for-bit image.** Capture the whole device (every sector, including unallocated space), not just files. Prefer a forensic format (E01) that stores metadata and hashes, or raw (dd) when needed:
   ```
   # forensic format with built-in hashing and metadata
   ewfacquire /dev/sdX
   # or raw with a forensic dd that hashes as it reads
   dc3dd if=/dev/sdX of=evidence.img hash=sha256 log=acquire.log
   ```
4. **Hash the source and the image**, and confirm they match. This is the mathematical proof the copy is faithful — it's the step that makes the image admissible:
   ```
   sha256sum /dev/sdX        # source (through the write blocker)
   sha256sum evidence.img    # image — must equal the source hash
   ```
5. **Verify the image independently** after acquisition (re-hash the image) and record all hashes in the case notes. Many tools compute the hash during acquisition and again on verification — both should agree.
6. **Work only on a copy of the image.** Make a working duplicate for analysis and keep the master image read-only and stored securely. If your working copy's hash ever diverges, you go back to the master.
7. **Maintain chain of custody** throughout: every person who handles the evidence, when, and why, in an unbroken log.

### Cheatsheet

```bash

ewfacquire /dev/sdX
guymager                      # GUI, E01/raw, hashes automatically

dc3dd if=/dev/sdX of=evidence.img hash=sha256 log=acquire.log

sha256sum /dev/sdX
sha256sum evidence.img

ewfverify evidence.E01
```

### Reading the acquisition

- **Source hash == image hash** is the whole ballgame — equal hashes prove the copy is bit-for-bit faithful. Record both.
- **A hash mismatch** means the acquisition failed or the source changed during capture (a failing drive, or no write blocker) — do not proceed to analysis; re-acquire and investigate why.
- **A drive throwing read errors** may need specialist handling; forcing it can worsen data loss. Note bad sectors and consider imaging tools built for failing media.
- **Any gap in the custody log** is what opposing counsel attacks — an unexplained handoff can invalidate otherwise perfect evidence.

### Getting it right (the practice)

- **Write blocker, always.** The fastest way to destroy a case is to mount the original read-write and let the OS touch timestamps. Block writes to the source.
- **Image the whole device**, including unallocated space and slack — deleted data and artifacts live there, and file-level copies miss them.
- **Hash at acquisition and verify after**, storing the values with the case. The hash is your integrity proof; without it the image is just a file.
- **Keep a pristine master** read-only and analyse a working copy, so you can always return to a verified original.
- **Document continuously** — condition, serials, hashes, every handoff. In forensics, undocumented is unreliable.

### Pitfalls

- **Skipping the write blocker.** Booting or mounting the evidence read-write alters it; the integrity argument is gone before you start.
- **File-copy instead of bit-for-bit.** Copying files misses deleted data, slack space, and metadata — the parts that often matter most.
- **No hash, or hashing only once.** Without matching source/image hashes you can't prove the copy is faithful. Compute and verify.
- **Analysing the master image.** Work on a duplicate; keep the original verifiable and untouched.
- **Broken chain of custody.** A single unexplained gap can sink admissibility no matter how clean the technical work was.

### References

- NIST SP 800-86 (Guide to Integrating Forensic Techniques into Incident Response)
- SWGDE best practices for computer forensics
- The Sleuth Kit / libewf documentation
- ACPO / ISO 27037 guidelines on digital evidence handling

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.