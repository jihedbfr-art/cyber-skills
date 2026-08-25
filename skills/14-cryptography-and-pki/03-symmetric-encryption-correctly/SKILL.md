---
format: "v2"
name: "symmetric-encryption-correctly"
title: "Symmetric Encryption Correctly"
title_fr: "Bien utiliser le chiffrement symétrique"
description: "Use when encrypting data with a symmetric cipher — using AEAD, handling nonces and keys correctly, and avoiding the mode and reuse mistakes that silently break confidentiality."
description_fr: "À utiliser pour chiffrer des données avec un algorithme symétrique — recourir à l'AEAD, gérer correctement les nonces et les clés, et éviter les erreurs de mode ou de réutilisation qui compromettent silencieusement la confidentialité."
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

Symmetric encryption looks simple — pick AES, encrypt — but the mistakes are subtle and silent: ECB mode that leaks structure, encryption without integrity that enables tampering, or a reused nonce that catastrophically breaks GCM. The ciphertext looks fine in every case; the security is gone. This skill covers using symmetric encryption the way it's meant to be used so confidentiality *and* integrity actually hold.

### When to use it

Any time you encrypt data at rest or in a protocol you're building, or review code that does. It's the concrete "how" behind the choosing-the-right-primitive guidance, focused on getting symmetric encryption right in practice.

### The rules

1. **Use AEAD — authenticated encryption — not a bare cipher.** AEAD (AES-GCM, ChaCha20-Poly1305) provides confidentiality *and* integrity in one construction, so tampering is detected. Encryption without authentication (bare CBC/CTR) lets an attacker modify ciphertext undetected, enabling padding-oracle and bit-flipping attacks. This is the single most important rule.
2. **Never use ECB.** ECB encrypts identical plaintext blocks to identical ciphertext blocks, leaking structure (the infamous "ECB penguin"). It's a finding wherever it appears.
3. **Handle nonces/IVs correctly — this is where GCM breaks catastrophically.** A nonce must be unique per key. **Reusing a nonce with the same key in GCM is catastrophic** — it can leak plaintext and, worse, the authentication key, breaking integrity for all messages under that key. Let the library generate nonces, use a random or properly-managed counter nonce, and never reuse one.
4. **Use strong keys from a CSPRNG**, of the right length (AES-256 = 32 bytes), and manage them properly (see the key-management skill) — encryption is only as strong as the key and its protection.
5. **Bind associated data where relevant** — AEAD lets you authenticate (but not encrypt) additional context (headers, IDs) so it can't be swapped; use the AAD parameter for data that must be tamper-evident alongside the ciphertext.
6. **Prefer a high-level library over assembling primitives** — libsodium (`crypto_secretbox`/`crypto_aead`) and equivalents make the safe choices for you and are hard to misuse. Hand-assembling cipher + mode + MAC is where the bugs live.

### Cheatsheet

```
USE (AEAD — confidentiality + integrity together)
  AES-256-GCM   |  ChaCha20-Poly1305   |  libsodium crypto_secretbox/aead
AVOID
  ECB (always — leaks structure)
  unauthenticated CBC/CTR alone (no integrity -> padding oracle, bit-flip)
  hand-assembling cipher+mode+MAC (encrypt-then-MAC) unless you must

nonce/IV rules (GCM breaks CATASTROPHICALLY on reuse)
  unique per key, ALWAYS ; let the library generate it ; NEVER reuse with same key
  reused GCM nonce -> leaks plaintext AND the auth key

keys: CSPRNG, correct length (AES-256 = 32 bytes), managed (see key-management)
AAD: authenticate context (headers/IDs) that must be tamper-evident, not encrypted
prefer: a high-level library that makes the safe choices for you
```

### Reading the code/design

- **ECB mode** = leaks plaintext structure; visible as repeating ciphertext for repeating plaintext. A clear finding — replace with AEAD.
- **Encryption without a MAC/AEAD** (bare CBC/CTR) = no integrity; tampering undetected, padding-oracle and bit-flipping attacks apply. Use AEAD (or encrypt-then-MAC if you truly must hand-build).
- **A reused or fixed nonce/IV with GCM** = catastrophic — leaks plaintext and the auth key. Among the most severe crypto bugs; the nonce must be unique per key.
- **A hardcoded or weak key, or one from a non-CSPRNG** = the encryption is only as strong as the key; a predictable key defeats it entirely.
- **Hand-assembled cipher + mode + MAC** = high bug risk (wrong order, mismatched keys, timing leaks); prefer a vetted AEAD library.
- **AES-256-GCM (or ChaCha20-Poly1305) via a good library, unique nonces, CSPRNG keys, AAD for context** = correct.

### The fix / best practice

- **Always use AEAD** (AES-256-GCM or ChaCha20-Poly1305) so integrity comes with confidentiality — replace ECB and unauthenticated modes.
- **Guarantee nonce uniqueness per key** — let the library generate nonces and never reuse one with the same key; rotate keys before nonce space is at risk for counter-based schemes.
- **Generate keys from a CSPRNG**, at full length, and manage them properly (key-management skill).
- **Use AAD** to bind context that must be tamper-evident.
- **Prefer a high-level library** (libsodium/NaCl) that makes these choices safely rather than assembling primitives yourself.
- Combine with proper key storage/rotation (key-management, secrets-at-rest skills).

### Pitfalls

- **Encrypting without authenticating.** Confidentiality without integrity is broken in practice (padding oracles, tampering). AEAD is the fix; bare CBC/CTR is a finding.
- **Nonce reuse with GCM.** Catastrophic — it can leak the authentication key, not just one message. The most dangerous symmetric-crypto mistake; ensure uniqueness.
- **ECB anywhere.** It leaks structure; there's essentially no correct use for it.
- **Rolling your own construction.** Wrong MAC order, key reuse across encrypt/MAC, and timing bugs live here. Use a vetted AEAD library.
- **Weak/predictable keys.** The strongest cipher is undone by a bad key; use a CSPRNG and manage keys well.

### References

- OWASP Cryptographic Storage Cheat Sheet
- libsodium documentation (crypto_secretbox / crypto_aead)
- NIST SP 800-38D (GCM) — nonce uniqueness requirements
- The choosing-the-right-primitive, key-management, and secrets-at-rest skills; CWE-327, CWE-329 (nonce reuse)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.