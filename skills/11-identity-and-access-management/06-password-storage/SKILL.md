---
format: "v2"
name: "password-storage"
title: "Password Storage"
title_fr: "Stockage des mots de passe"
description: "Use when implementing or reviewing how user passwords are stored — choosing a slow password hash, setting its parameters, and avoiding the storage mistakes that make a breach catastrophic."
description_fr: "À utiliser pour mettre en place ou auditer le stockage des mots de passe utilisateurs : choisir une fonction de hachage lente adaptée, régler ses paramètres, et éviter les erreurs de stockage qui transforment une fuite de données en catastrophe."
domain: "11-identity-and-access-management"
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

Databases get breached. What decides whether that's a disaster or a shrug is how the passwords were stored. Done right, a stolen password database yields almost nothing usable; done wrong (plaintext, MD5, unsalted SHA), attackers crack most of it in hours and reuse it everywhere. This skill covers storing passwords so a breach stays survivable.

### When to use it

Building authentication, reviewing an existing implementation, or planning a migration off a weak scheme. It's the concrete "how" behind the crypto domain's choosing-the-right-primitive guidance, focused on passwords specifically.

### The rules

1. **Use a slow, salted password hash — a KDF built for passwords**, not a general-purpose fast hash:
   - **Argon2id** — first choice (memory-hard, resists GPU/ASIC cracking).
   - **scrypt** — also memory-hard, good.
   - **bcrypt** — still fine and widely available; cap-aware (see pitfalls).
   - **Never** MD5, SHA-1, SHA-256, or any plain fast hash — they're built to be fast, which is exactly what helps the attacker.
2. **Per-password random salt** (the modern KDFs handle this for you) so identical passwords hash differently and precomputed rainbow tables are useless.
3. **Tune the cost/work factor** so hashing takes a deliberate fraction of a second on your hardware — slow enough to cripple offline cracking, fast enough not to DoS your own login.
4. **Consider a pepper** (a secret, kept outside the database — in a config secret or HSM) added to the input, so DB-only theft still leaves the attacker missing a piece.

### Procedure (reviewing)

1. **Find where passwords are hashed** and identify the algorithm. Grep for hashing calls; the stored hash format often gives it away (`$2b$` = bcrypt, `$argon2id$` = Argon2, a bare 32/64 hex string = a raw fast hash — bad).
2. **Flag anything fast or unsalted**: plaintext, `md5(pw)`, `sha256(pw)`, unsalted hashes. These are the critical findings — a breach cracks them trivially.
3. **Check the parameters** on an otherwise-good KDF: bcrypt cost factor high enough (≥12 as a common baseline, tuned to hardware), Argon2 memory/iterations sensible. A modern hash with weak parameters is much weaker than it looks.
4. **Confirm salting** is per-password and random (automatic with Argon2/bcrypt/scrypt; a concern only in hand-rolled schemes).
5. **Check the login path** for timing leaks and that comparison is done via the KDF's verify function, not a naive string compare.
6. **Plan migration** if the scheme is weak (see the fix) — you rarely can un-hash to re-hash, so migration is upgrade-on-next-login.

### Cheatsheet

```
use:    Argon2id  (or scrypt / bcrypt)      per-password random salt (automatic)
avoid:  plaintext, md5, sha1, sha256, any unsalted/fast hash

identify a stored hash
  $argon2id$...   Argon2id  (good)
  $2b$12$...      bcrypt, cost 12 (good; number = cost)
  $2b$06$...      bcrypt, cost 6  (too low — weak)
  a1b2c3... (bare 32/64 hex)   raw MD5/SHA — BAD

tune (aim ~0.2-0.5s per hash on your hardware)
  bcrypt: cost >= 12 (adjust up over time)
  argon2id: set memory (e.g. 19+ MiB), iterations, parallelism per guidance

migration (can't reverse a hash): wrap old hash or re-hash on next login
```

### Reading the review

- **Plaintext or a fast/unsalted hash** = critical; a breach exposes usable passwords that get reused against every other service the users have. This is the finding that turns one breach into many.
- **A good KDF with weak parameters** (bcrypt cost 6, trivial Argon2 settings) = weaker than it appears; crackable far faster than intended. Raise the cost.
- **A single global salt or no salt** (in hand-rolled code) = rainbow tables and cross-user cracking apply; effectively unsalted.
- **Naive string comparison of hashes** = a timing side channel; use the library's verify.
- **Argon2id with tuned parameters and per-password salt** = the good state; a breach yields little.

### The fix

- **Store with Argon2id** (or scrypt/bcrypt), per-password random salt, parameters tuned so one hash takes a noticeable fraction of a second — then revisit the cost periodically as hardware improves.
- **Migrate off weak schemes** without needing the plaintext: on each successful login, verify against the old hash then re-hash with the new KDF and store that; or immediately wrap existing hashes (`argon2(old_hash)`) and unwrap-then-rehash over time.
- **Add a pepper** stored outside the database for defence in depth.
- **Enforce this at one place** (a single hashing/verify utility) so no code path stores a password differently.
- Pair with the auth skills: breached-password screening, rate limiting, and MFA reduce the damage even further.

### Pitfalls

- **A fast hash "because it's still a hash".** SHA-256 on passwords is cracked at billions/sec. Slowness is the whole point; use a password KDF.
- **Weak parameters on a good algorithm.** bcrypt cost 6 or minimal Argon2 settings undo the algorithm's strength.
- **bcrypt's 72-byte input limit.** Long passwords are silently truncated; pre-hash (e.g. SHA-256 then bcrypt) or use Argon2 if that matters.
- **Rolling your own salting/hashing.** Use the library's KDF and verify functions; hand-rolled schemes are where unsalted/timing bugs live.

### References

- OWASP Password Storage Cheat Sheet
- NIST SP 800-63B (memorized secret verifiers)
- Argon2 / bcrypt / scrypt documentation
- CWE-256, CWE-916 (weak password hashing)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.