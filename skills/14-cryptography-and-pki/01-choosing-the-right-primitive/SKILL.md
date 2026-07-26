---
name: choosing-the-right-primitive
domain: 14-cryptography-and-pki
description: Use when you need to pick a cryptographic primitive for a task — hashing, encryption, signing, password storage — and want the safe modern default instead of a broken one.
difficulty: beginner
tags: [cryptography, primitives, hashing, encryption, best-practice]
tools: []
---

## Purpose

Most crypto bugs aren't broken algorithms — they're the wrong tool for the job. MD5 for passwords, ECB mode, a hash where you needed a KDF, encryption where you needed a signature. This skill is a decision guide: match the task to the right primitive and the current safe default, so you don't reach for something that was fine in 2010 and is a finding today.

The rule underneath all of it: don't invent crypto, and don't configure it from a decade-old tutorial. Use a vetted library with modern defaults.

## When to use it

Any time you're about to hash, encrypt, sign, or store a secret and aren't certain which primitive fits — or when reviewing code that makes one of those choices. Pair it with the code-review crypto-misuse skill when auditing existing code.

## Match the task to the primitive

**Integrity / fingerprinting (not passwords):**
Use **SHA-256** (or SHA-3/BLAKE2). Avoid MD5 and SHA-1 — both are broken for collision resistance.

**Password storage:**
Use a **slow password hash / KDF**: **Argon2id** (first choice), or **scrypt** / **bcrypt**. Never a plain fast hash (SHA-256, MD5) — those are built to be fast, which is exactly what you don't want against cracking. See the password-hashing skill for parameters.

**Symmetric encryption (confidentiality + integrity):**
Use an **AEAD** cipher: **AES-256-GCM** or **ChaCha20-Poly1305**. Avoid ECB always, and avoid unauthenticated CBC/CTR alone (they encrypt without detecting tampering). AEAD gives you both in one.

**Message authentication (shared secret):**
Use **HMAC-SHA-256**. Don't roll your own "hash(secret + message)" — that's the construction HMAC exists to replace safely.

**Digital signatures (public/private key):**
Use **Ed25519** (or ECDSA P-256, or RSA-2048+ with PSS padding). Prefer Ed25519 for new designs — fast and hard to misuse.

**Key derivation from a shared secret / key stretching:**
Use **HKDF** for deriving keys from high-entropy material; a password KDF (Argon2/scrypt) when the input is a low-entropy password.

**Randomness (keys, tokens, nonces, salts):**
Use a **cryptographically secure RNG** (`/dev/urandom`, `secrets` in Python, `crypto.randomBytes` in Node). Never `rand()`/`Math.random()`/`java.util.Random` for anything security-relevant.

## Cheatsheet

```
task                        use                          avoid
--------------------------  ---------------------------  -----------------------
integrity / hashing         SHA-256, SHA-3, BLAKE2       MD5, SHA-1
password storage            Argon2id, scrypt, bcrypt     any fast hash, plain SHA
symmetric encryption        AES-256-GCM, ChaCha20-Poly   ECB, unauthenticated CBC
message auth (MAC)          HMAC-SHA-256                 homemade hash(secret+msg)
signatures                  Ed25519, ECDSA P-256, RSA-PSS  RSA PKCS#1v1.5 (new use)
key derivation (secret)     HKDF                         raw hash truncation
key derivation (password)   Argon2id / scrypt            HKDF on a password
random values               CSPRNG (secrets/urandom)     rand(), Math.random()
```

## Reading a design/codebase for this

- **A fast hash on passwords** (`sha256(password)`, `md5(...)`) is a finding — cracking is trivial at scale. Move to Argon2id.
- **ECB mode** anywhere shows as repeating ciphertext blocks and leaks structure — replace with AEAD.
- **Encryption without a MAC/AEAD** means tampering isn't detected; padding-oracle and bit-flipping attacks apply. Use GCM or add HMAC (encrypt-then-MAC).
- **`Math.random()`/`rand()` generating a token, key, or salt** is predictable — swap to a CSPRNG.
- **A hand-rolled MAC** (`hash(secret + msg)`) is length-extension-prone — use HMAC.

## The safe defaults (the "fix")

When in doubt, these are the modern choices to standardise on:

- Hash: **SHA-256**
- Passwords: **Argon2id**
- Symmetric: **AES-256-GCM** (or ChaCha20-Poly1305 where AES hardware isn't available)
- MAC: **HMAC-SHA-256**
- Signatures: **Ed25519**
- KDF: **HKDF** (from keys) / **Argon2id** (from passwords)
- Randomness: the platform **CSPRNG**

Use a maintained library that implements these with safe defaults (libsodium/NaCl is hard to misuse), keep the library updated, and let it manage nonces/IVs where it can.

## Pitfalls

- **Encryption when you needed a signature (or vice versa).** Confidentiality and authenticity are different goals; AEAD covers both for symmetric, signatures for asymmetric.
- **A hash where a KDF belongs.** Passwords need slowness; fingerprints need speed. Don't swap them.
- **Reusing a nonce/IV with GCM.** Catastrophic for GCM — let the library handle nonce generation, never reuse one with the same key.
- **Copying config from an old tutorial.** Yesterday's "secure" (SHA-1, RSA-1024, CBC) is today's finding. Check the default is current.

## References

- OWASP Cryptographic Storage Cheat Sheet
- NIST SP 800-175B / SP 800-131A (approved algorithms and transitions)
- libsodium documentation
- CWE-327 (Broken/Risky Crypto Algorithm), CWE-326 (Inadequate Encryption Strength)
