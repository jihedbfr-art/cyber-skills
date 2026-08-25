---
format: "v2"
name: "crypto-misuse-review"
title: "Crypto Misuse Review"
title_fr: "Revue des mauvais usages de la cryptographie"
description: "Use when reviewing code that encrypts, hashes, signs, or generates randomness — finding weak algorithms, misused primitives, and hand-rolled crypto before they ship."
description_fr: "À utiliser lors de la revue de code qui chiffre, hache, signe ou génère de l'aléatoire — pour repérer les algorithmes faibles, les primitives mal utilisées et la cryptographie maison avant leur mise en production."
domain: "10-secure-code-review"
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

Almost nobody breaks crypto by attacking the math. They break it because the code used ECB, reused a nonce, hashed passwords with SHA-256, or seeded a "random" token with the current time. This skill is a review pass for the recognisable misuse patterns — you don't need to be a cryptographer to catch the ones that actually get exploited.

### When to use it

Any change touching encryption, password storage, token/session generation, signing, or "generate a secret." Also worth a periodic sweep — crypto misuse tends to get copied from an old file into a new one.

### The patterns, worst first

- **Passwords through a fast hash.** MD5, SHA-1, SHA-256, SHA-512 for password storage. Fast is the problem — it makes cracking cheap. Password storage wants a slow, salted KDF: bcrypt, scrypt, Argon2id, or PBKDF2 with a high iteration count. (There's a dedicated skill for this in domain 11; here you just need to spot the fast hash and stop.)
- **ECB mode.** `AES/ECB/...`, or any encryption where identical plaintext blocks produce identical ciphertext. It leaks structure. Authenticated modes (GCM, or ChaCha20-Poly1305) are the default answer.
- **Static or reused IV/nonce.** A hardcoded IV, an all-zero IV, or a counter that resets. With GCM, nonce reuse under the same key is catastrophic — it can leak the auth key. Look for the IV being a constant or derived deterministically.
- **`Math.random()` / `rand()` for anything security-relevant.** Tokens, password-reset codes, session ids, salts. These PRNGs are predictable. Security randomness needs a CSPRNG: `SecureRandom`, `crypto.randomBytes`, `secrets`, `/dev/urandom`.
- **Hardcoded keys / IVs / salts.** A key in source is not a key. (Overlaps with the secrets-in-code skill — flag from both angles.)
- **Encrypt-without-authenticate.** CBC without a MAC invites padding-oracle and bit-flipping attacks. Prefer AEAD; if you see raw CBC, ask where the integrity check is.
- **Home-grown crypto.** Custom "encryption," XOR obfuscation dressed up as security, a bespoke signature scheme. Treat any hand-rolled primitive as broken until a real one replaces it.
- **Weak or skipped verification.** Signature/JWT verification that catches the exception and continues, `verify=False`, accepting `alg: none`. (JWT specifics live in domain 04.)

### Procedure

1. Grep the primitives and modes (cheatsheet).
2. Sort hits by blast radius: password hashing and key handling first, then modes/IVs, then randomness.
3. For each, name the property that should hold (confidentiality, integrity, unpredictability, slow-to-crack) and check the code actually provides it.
4. Follow keys and IVs to their origin — constant, config, derived, or a KDF? Constants are findings.
5. Don't try to "fix" a hand-rolled scheme by patching it. The finding is *replace it with a standard library primitive.*

### Cheatsheet

```bash
rg -n 'MD5|SHA-?1|MessageDigest\.getInstance\("(MD5|SHA-1|SHA-256)"'
rg -n 'AES/ECB|Cipher\.getInstance\("AES"\)|DES|RC4|/CBC/'
rg -n 'Math\.random|new Random\(|rand\(\)|mt_rand|random\.random\('
rg -n 'IvParameterSpec\(|SecretKeySpec\(|byte\[\] *(key|iv) *='
rg -n 'verify=False|alg.{0,3}none|InsecureRequestWarning|checkServerIdentity'
```

### Reading it

- **A fast hash on a password** → finding, regardless of salt. Salt doesn't fix speed.
- **The same MD5 on a file checksum or cache key** → fine. Judge by *what it protects*, not the algorithm name alone.
- **`SecureRandom` seeded manually** (`new SecureRandom(seed)` with a known seed) → back to predictable; flag it.
- **GCM with a nonce that isn't unique per message** → critical; note it specifically.
- **A `catch` around signature verification that logs and proceeds** → the verification is theatre.

### The fix

Reach for the platform's vetted library and its safe defaults: Argon2id/bcrypt for passwords, AES-GCM or ChaCha20-Poly1305 for encryption, a CSPRNG for anything unpredictable, keys from a secrets manager or KMS. The reviewer's rule of thumb: if the code *chooses* a mode, IV, or hash by hand, that's where the bug is — safe-by-default APIs don't make you choose.

### Pitfalls

- **Algorithm-name matching without context.** MD5 for a non-security checksum isn't a bug. Ask what property it's guarding.
- **Accepting a salt as sufficient for a fast hash.** It isn't — the KDF's slowness is the point.
- **Missing the IV.** People check the cipher and forget the nonce, which is where GCM actually dies.
- **Patching hand-rolled crypto.** Replace, don't repair.

### References

- OWASP Cryptographic Storage Cheat Sheet
- CWE-327 (broken/risky algorithm), CWE-329 (missing/predictable IV), CWE-338 (weak PRNG), CWE-916 (weak password hash)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.