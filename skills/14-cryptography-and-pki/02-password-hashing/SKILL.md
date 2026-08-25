---
format: "v2"
name: "password-hashing"
title: "Password Hashing"
title_fr: "Hachage des mots de passe"
description: "Use when choosing and tuning a password hash — Argon2, scrypt, or bcrypt with parameters that stay expensive to crack — so a stolen database yields little."
description_fr: "À utiliser pour choisir et régler un algorithme de hachage de mots de passe — Argon2, scrypt ou bcrypt avec des paramètres qui restent coûteux à casser — afin qu'une base de données volée ne livre presque rien."
domain: "14-cryptography-and-pki"
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

This is the cryptographic detail behind storing passwords: which algorithm, and — the part that decides everything — what parameters. A slow password hash with weak parameters is barely slower to crack than a fast hash. This skill focuses on the algorithm-and-tuning choice; the IAM `password-storage` skill covers the surrounding storage and migration workflow, and they're deliberately complementary.

### When to use it

Implementing password storage, or reviewing whether an existing scheme's *parameters* (not just its algorithm) actually resist cracking. The common failure isn't picking MD5 anymore — it's picking bcrypt with a cost factor from 2012.

### Choosing and tuning

1. **Pick a memory-hard KDF where you can.** In order of preference:
   - **Argon2id** — first choice; memory-hard, resists GPU/ASIC cracking, tunable across memory, iterations, and parallelism.
   - **scrypt** — also memory-hard, good where Argon2 isn't available.
   - **bcrypt** — still acceptable and ubiquitous, but not memory-hard (so more GPU-crackable at scale) and has a 72-byte input limit.
   Avoid any fast hash (SHA-256, MD5) and PBKDF2 unless a compliance requirement forces it (PBKDF2 is not memory-hard; if required, use a high iteration count with SHA-256).
2. **Tune the cost so one hash takes a deliberate fraction of a second on your hardware** — roughly 0.25–0.5s is a common target, balancing crack-resistance against login latency and your server's capacity under load. The right parameters depend on *your* hardware, so measure:
   - **Argon2id:** set memory (e.g. tens of MiB, higher is stronger), iterations (time cost), and parallelism per current OWASP guidance, then measure.
   - **bcrypt:** cost factor (work factor) — 12 is a common current baseline; raise it as hardware improves and re-measure.
3. **Let the library handle the salt** — modern KDFs generate a unique random salt per password automatically and embed it in the output. Don't hand-roll salting.
4. **Consider a pepper** — a secret added to the input, stored outside the database (config/HSM/KMS), so a database-only breach still leaves the attacker missing a component.
5. **Re-tune over time.** Parameters that were strong three years ago are weaker against today's hardware; revisit the cost periodically and upgrade on next login.

### Cheatsheet

```
choose (memory-hard preferred)
  Argon2id  (1st)   memory + iterations + parallelism, tunable, GPU/ASIC-resistant
  scrypt    (2nd)   memory-hard
  bcrypt    (ok)    ubiquitous, NOT memory-hard, 72-byte input cap
  avoid: MD5/SHA-* (fast) ; PBKDF2 only if compliance forces it (high iterations)

tune to ~0.25-0.5s per hash ON YOUR HARDWARE (measure!)
  Argon2id: memory (tens of MiB+), time cost, parallelism (per OWASP guidance)
  bcrypt:   cost factor >= 12 (raise over time)

salt: per-password random, AUTO by the KDF (don't hand-roll)
pepper: optional secret kept OUTSIDE the DB (config/HSM/KMS) — defence in depth
re-tune periodically ; upgrade parameters on next login
```

### Reading a scheme

- **A fast hash (SHA-256/MD5) on passwords** = cracked at billions/sec; a breach exposes usable passwords. The primary finding — move to a memory-hard KDF.
- **A good algorithm with weak parameters** (bcrypt cost 6, minimal Argon2 memory) = far weaker than it looks; crackable much faster than intended. Tuning is the real control, so under-tuned is a real finding.
- **bcrypt on long passwords** = silent truncation past 72 bytes; if long passphrases matter, pre-hash or use Argon2.
- **PBKDF2 with a low iteration count** = fast-crackable; not memory-hard, so weak against GPUs unless iterations are very high.
- **Argon2id tuned to ~0.3s with per-password salt** = the strong state; a breach yields little.

### The fix / best practice

- **Standardise on Argon2id** (or scrypt/bcrypt) with parameters tuned by measurement on your hardware to a deliberate fraction of a second.
- **Never use a fast hash** for passwords; slowness is the entire point.
- **Rely on the library's per-password salting**, and consider a database-external pepper for defence in depth.
- **Revisit parameters periodically** and increase cost as hardware advances, upgrading stored hashes on next successful login (migration mechanics are in the IAM password-storage skill).
- **Validate the choice** by attempting to crack a sample with hashcat at realistic settings — if it falls fast, your parameters are too low.

### Pitfalls

- **Right algorithm, wrong parameters.** The modern failure mode — bcrypt/Argon2 with weak cost is only marginally better than a fast hash. Tune and measure.
- **Copying parameters from an old tutorial.** Yesterday's "strong" cost is today's weak; base it on current guidance and your hardware.
- **Hand-rolling salts.** Use the KDF's built-in per-password salting; homemade salting is where reuse and predictability bugs come from.
- **Ignoring bcrypt's 72-byte limit.** Long passwords get truncated silently; pre-hash or use Argon2 if that matters.
- **Never re-tuning.** Hardware improves; static parameters weaken over time. Revisit.

### References

- OWASP Password Storage Cheat Sheet (current Argon2/bcrypt/scrypt parameters)
- Argon2 RFC 9106, and scrypt / bcrypt documentation
- The IAM password-storage skill (storage workflow and migration) and choosing-the-right-primitive skill
- CWE-916 (use of password hash with insufficient computational effort)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.