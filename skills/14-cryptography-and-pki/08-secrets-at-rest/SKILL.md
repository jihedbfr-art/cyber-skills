---
format: "v2"
name: "secrets-at-rest"
title: "Secrets At Rest"
title_fr: "Secrets au repos"
description: "Use when protecting stored secrets and sensitive data at rest — envelope encryption and secret managers — so a database or disk breach doesn't hand over plaintext."
description_fr: "À utiliser pour protéger les secrets et données sensibles stockés — chiffrement enveloppe et gestionnaires de secrets — afin qu'une compromission de base de données ou de disque ne livre pas de données en clair."
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

Data and secrets stored on disk or in a database will eventually be exposed to someone they shouldn't be — a breach, a backup that leaks, a decommissioned drive. Encryption at rest is what makes that exposure harmless: the attacker gets ciphertext, not plaintext. But it only works if the *key* isn't sitting right next to the data. This skill covers protecting secrets and sensitive data at rest so that stealing the storage doesn't mean stealing the contents.

### When to use it

Storing anything sensitive — application secrets (API keys, DB passwords), user PII, or confidential data — in a database, on disk, or in backups. It ties together the encryption and key-management skills into the specific problem of at-rest protection.

### The two problems

- **Secrets** (credentials the app needs) — API keys, database passwords, tokens. These should live in a **secret manager**, fetched at runtime, not in code/config/env.
- **Sensitive data** (user PII, confidential records) — encrypted in the database/storage using **envelope encryption** with keys the storage layer can't decrypt on its own.

### Procedure

1. **Put application secrets in a secret manager.** Credentials the app needs at runtime belong in Vault / cloud Secrets Manager / KMS — access-controlled, audited, rotatable — and fetched at runtime, never hardcoded (the credential-hygiene and secrets-in-code skills). This is the fix for the "password in the config file" problem.
2. **Encrypt sensitive data with envelope encryption.** Don't encrypt everything with one static key sitting near the data. Instead: a KMS master key encrypts a data key, the data key encrypts the data; the master key never leaves the KMS. A database breach yields encrypted data and an encrypted data key — useless without KMS access:
   ```
   # envelope: KMS master key (never leaves KMS) -> encrypts data key -> encrypts data
   ```
3. **Keep the key out of reach of the data's attacker.** The whole point: if the same breach that exposes the data also exposes the key, encryption at rest achieved nothing. The key must be protected by a different boundary (KMS/HSM access control), not stored in the same database or config.
4. **Use full-disk / transparent DB encryption as a baseline, but understand its limit** — TDE/disk encryption protects against physical theft of the drive, but not against an attacker who has access to the running database (they see decrypted data). For sensitive fields, application-level/envelope encryption adds protection that survives a live-database compromise.
5. **Protect backups too** — backups are a frequent leak source and often forgotten; ensure they're encrypted with keys managed the same way, or a stolen backup undoes all the at-rest protection.
6. **Rotate keys and re-encrypt** per the key-management lifecycle, and enable access logging so key use for decryption is auditable.

### Cheatsheet

```
two problems, two answers
  app secrets (API keys, DB passwords)  -> SECRET MANAGER (Vault/KMS/SM),
                                            fetched at runtime, never hardcoded
  sensitive data (PII, records)         -> ENVELOPE ENCRYPTION via KMS

envelope encryption
  KMS master key (never leaves KMS) -> encrypts data key -> encrypts the data
  breach gets ciphertext + encrypted data key = useless without KMS access

the core rule: the KEY must not be reachable by the DATA's attacker
  (key beside the data in DB/config = encryption at rest achieved nothing)

layers
  full-disk / TDE   -> protects the STOLEN DRIVE, not a live-DB compromise
  app/envelope enc  -> protects sensitive fields even vs a compromised running DB
  BACKUPS           -> encrypt them too (frequent, forgotten leak source)
```

### Reading the setup

- **Secrets in code/config/env vars** = plaintext credentials anyone with the repo/host obtains; "encryption at rest" on the DB doesn't help if the DB password is in the config. Move to a secret manager.
- **Data encrypted with a key stored beside it** (in the same DB, in config) = the same breach gets both; the encryption is cosmetic. The key must be behind a different boundary (KMS).
- **Only full-disk/TDE encryption on sensitive data** = protects against drive theft but not a live-database compromise (SQLi, a compromised app sees plaintext). Add app/envelope encryption for the sensitive fields.
- **Unencrypted backups** = a stolen backup exposes everything the at-rest encryption was meant to protect. Encrypt backups with managed keys.
- **Secret manager for secrets + envelope encryption with KMS-held keys + encrypted backups** = at-rest protection that actually survives a breach.

### The fix / best practice

- **Application secrets → secret manager**, fetched at runtime, access-controlled and rotatable; never hardcoded.
- **Sensitive data → envelope encryption** with a KMS master key that never leaves the KMS, so a storage breach yields only ciphertext.
- **Keep the key behind a different boundary than the data** — the defining principle of meaningful at-rest encryption.
- **Layer disk/TDE (drive theft) with app/envelope encryption (live-DB compromise)** for sensitive fields.
- **Encrypt backups** with the same key discipline.
- **Rotate keys, re-encrypt, and log decryption access** (key-management skill).

### Pitfalls

- **The key stored with the data.** The defining failure — if the breach that gets the data also gets the key, at-rest encryption protects nothing. Put the key in a KMS behind separate access control.
- **Secrets in config/env "because it's internal".** Internal systems get breached; plaintext credentials are then handed over. Use a secret manager.
- **Relying on disk/TDE alone for sensitive data.** It stops drive theft, not a compromised running database; add application-level encryption for sensitive fields.
- **Forgetting backups.** Unencrypted backups are a common, overlooked leak that bypasses all your other at-rest protection.
- **One static key forever.** No rotation means a leaked key exposes everything indefinitely; rotate and re-encrypt.

### References

- OWASP Cryptographic Storage and Secrets Management Cheat Sheets
- Cloud KMS envelope-encryption documentation; HashiCorp Vault
- The key-management, kms-and-secrets-management (cloud), and secrets-in-code skills
- CWE-311 (missing encryption of sensitive data), CWE-312 (cleartext storage)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.