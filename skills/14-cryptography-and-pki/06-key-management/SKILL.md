---
name: key-management
domain: 14-cryptography-and-pki
description: Use when managing cryptographic keys across their lifecycle — generation, storage, rotation, and destruction — because the key, not the algorithm, is where crypto usually fails.
difficulty: advanced
tags: [crypto, key-management, hsm, kms, rotation]
tools: [kms, vault]
---

## Purpose

Strong encryption with a poorly-managed key is weak encryption. The algorithm is rarely the problem — the key being generated weakly, stored in the clear, never rotated, or copied everywhere is. Key management is the unglamorous discipline that decides whether your crypto actually protects anything. This skill covers the key lifecycle — generation, storage, use, rotation, and destruction — and the systems (KMS/HSM) that do it right.

## When to use it

Any system that uses cryptographic keys for encryption, signing, or authentication (nearly all of them at scale), or reviewing whether keys are managed properly behind otherwise-good crypto. It underpins the encryption, secrets, and PKI skills — they all assume the keys are well-managed.

## The key lifecycle

1. **Generation** — keys must come from a **CSPRNG**, at the correct length for the algorithm. A predictable or weak key defeats everything downstream. Generate keys inside the system that will protect them (KMS/HSM) where possible, so the raw key never exists outside.
2. **Storage — the crux.** Keys must not sit in plaintext where the data they protect is. Store them in a dedicated key-management system:
   - **KMS** (cloud key management) — manages keys, enforces access policy, logs usage; the data key is encrypted by a master key that never leaves the service (envelope encryption).
   - **HSM** (hardware security module) — keys generated and used inside tamper-resistant hardware; the key never leaves in plaintext. For the highest-value keys.
   - **Secret managers** (Vault) — for keys/secrets applications fetch at runtime.
   Never hardcode keys in code, config, or images (the credential-hygiene and secrets-in-code skills).
3. **Access control** — who/what can use a key is as important as where it's stored. Scope key usage to the minimal set of principals, and log every use (a key any role can use protects data from outsiders, not insiders — the KMS skill's point).
4. **Rotation** — rotate keys periodically and on suspected compromise, so a leaked key has a bounded useful life and the volume of data under any one key is limited. Design for rotation from the start (see crypto-agility) — retrofitting it is painful.
5. **Separation** — separate keys by purpose and environment (don't use the prod key in dev, don't use one key for everything), so a compromise is contained.
6. **Destruction** — when a key is retired, destroy it securely so retired keys can't be recovered and misused; but retain the ability to decrypt data still encrypted under an old key until it's re-encrypted (rotation must handle in-flight data).

## Cheatsheet

```
the principle: the KEY, not the algorithm, is usually where crypto fails.

lifecycle
  generate  CSPRNG, correct length ; ideally inside the KMS/HSM (never leaves)
  store     KMS (envelope enc) | HSM (highest value, key never leaves hw) |
            secret manager (runtime) ; NEVER hardcoded in code/config/image
  access    least privilege on key USE + log every use (matters vs insiders)
  rotate    periodically + on compromise ; limits leaked-key lifetime & data-per-key
  separate  by purpose + environment (no one-key-for-everything, no prod key in dev)
  destroy   securely on retirement ; but keep ability to decrypt old data until re-encrypted

design for rotation from day one (retrofit = painful) -> see crypto-agility
```

## Reading the setup

- **Keys hardcoded in code/config/images** = plaintext keys where anyone with the repo or the image has them; the encryption protects nothing against that access. The most common and serious key-management failure.
- **Keys stored alongside the data they protect** = a single breach gets both ciphertext and key. Separate them into a KMS/HSM.
- **A weak or non-CSPRNG-generated key** = predictable, defeating the crypto regardless of algorithm strength.
- **One key for everything / prod keys in dev** = a single compromise is unbounded and reaches all environments. Separate by purpose and environment.
- **Keys never rotated** = a leaked key stays useful forever, and huge data volumes accumulate under one key. Rotate on a schedule and on compromise.
- **KMS/HSM storage, least-privilege logged access, rotation, separation** = well-managed keys; the crypto actually protects.

## The fix / best practice

- **Generate keys from a CSPRNG at correct length**, ideally inside a KMS/HSM so the raw key never exists outside protected boundaries.
- **Store keys in a KMS/HSM/secret manager**, never hardcoded and never beside the protected data.
- **Enforce least-privilege, logged access** to key usage — the key policy is a security control (KMS skill).
- **Rotate** periodically and on compromise, designing rotation in from the start (crypto-agility) and handling data encrypted under old keys.
- **Separate keys** by purpose and environment to contain compromise.
- **Destroy retired keys securely** while retaining decryption capability for still-encrypted data.

## Pitfalls

- **Hardcoded keys.** A plaintext key in code/config/image means the encryption protects nothing from anyone with that access. Use a KMS/secret manager.
- **Key stored with the data.** One breach yields both; separation is the whole point of a KMS.
- **No rotation, or no design for it.** A leaked key lives forever, and retrofitting rotation onto a system that assumed a single static key is painful. Design for it early.
- **One key for everything.** Maximises blast radius; separate by purpose and environment.
- **Weak generation.** A predictable key from a bad RNG defeats the strongest cipher. Use a CSPRNG, ideally inside the KMS/HSM.

## References

- NIST SP 800-57 (Key Management) and SP 800-130 (key management framework)
- Cloud KMS and HSM documentation; HashiCorp Vault
- The kms-and-secrets-management (cloud), symmetric-encryption, secrets-at-rest, and crypto-agility skills
- CWE-320 (key management errors), CWE-321 (hardcoded key)
