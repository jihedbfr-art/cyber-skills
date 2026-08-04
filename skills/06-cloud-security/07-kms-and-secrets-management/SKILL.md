---
name: kms-and-secrets-management
domain: 06-cloud-security
description: Use when managing encryption keys and secrets in the cloud — key policies, envelope encryption, and secret stores — so encrypted data and stored credentials stay actually protected.
difficulty: intermediate
tags: [cloud, kms, secrets, encryption, key-management]
tools: [aws-cli]
---

## Purpose

Cloud encryption is easy to turn on and easy to get subtly wrong. The data is encrypted, but the key policy lets the wrong principal decrypt it; or secrets sit in plaintext env vars while a managed secret store goes unused. This skill covers using cloud KMS and secret managers so that "encrypted" actually means protected — the access to the key matters as much as the encryption itself. AWS KMS/Secrets Manager are the examples; Azure Key Vault and GCP KMS follow the same principles.

## When to use it

Setting up encryption for data at rest, managing application secrets, or reviewing whether existing encryption and secret handling actually protect anything. Pairs with the credential-hygiene and S3 skills.

## Procedure

1. **Understand who can use the key, not just that data is encrypted.** In KMS, the **key policy** (plus IAM and grants) decides who can decrypt. Encryption is only as strong as that access control — a key any principal can use protects data from outsiders but not from a compromised insider role:
   ```
   aws kms get-key-policy --key-id <id> --policy-name default
   ```
2. **Scope key access to the principals that need it.** Separate keys by purpose/sensitivity and grant decrypt to the minimal set of roles. A single key used everywhere with broad access means one compromised role decrypts everything.
3. **Use envelope encryption correctly** — KMS encrypts a data key, the data key encrypts the data. Let the cloud SDK handle this; the point is that the master key never leaves KMS and rotation is manageable.
4. **Enable key rotation** where supported, and prefer customer-managed keys over the account-default key when you need control over the policy and rotation.
5. **Put secrets in a secret manager, not in code or plain config.** Application credentials, API keys, DB passwords belong in Secrets Manager / Key Vault / Secret Manager — fetched at runtime, access-controlled, auditable, and rotatable. Env vars and committed config are leak vectors (ties into credential-hygiene and secret-scanning).
6. **Audit access.** KMS decrypt calls and secret retrievals are logged (CloudTrail) — confirm they're captured, and alert on unusual decrypt patterns or access from unexpected principals.

## Cheatsheet

```bash
# who can DECRYPT? (the real question — not just "is it encrypted")
aws kms get-key-policy --key-id KEY --policy-name default
aws kms list-grants --key-id KEY
  -> scope decrypt to minimal roles; separate keys by sensitivity

# key hygiene
aws kms enable-key-rotation --key-id KEY          # rotate
prefer customer-managed keys (control policy + rotation) over default

# secrets: store them properly, fetch at runtime
aws secretsmanager get-secret-value --secret-id NAME
  -> NOT in code, NOT in plain env vars/config

# audit: KMS decrypt + secret retrieval are in CloudTrail -> alert on anomalies
```

## Reading the review

- **A key policy granting decrypt broadly** (a wildcard principal, or every role) = the encryption protects against outsiders but not a compromised insider role; access to the key is the weak link. Scope it down.
- **One key for everything** = a single compromised principal decrypts all data classes. Separate keys by purpose/sensitivity.
- **Secrets in plain env vars or committed config** = encryption at rest is undermined by credentials sitting in the open elsewhere. Move them to a secret manager.
- **No key rotation / account-default key used for sensitive data** = less control and a longer-lived key; prefer customer-managed keys with rotation.
- **Decrypt/secret-access not audited** = you can't detect misuse of the key; enable and alert on the CloudTrail events.

## The fix

- **Treat the key policy as the control.** Grant decrypt to the minimum principals, separate keys by data sensitivity, and review key policies like you review IAM — because that's what they are.
- **Use customer-managed keys with rotation** for sensitive data, so you control the policy and lifecycle.
- **Envelope encryption via the SDK** — master key stays in KMS, data keys encrypt the data.
- **Centralise secrets in a secret manager**, fetched at runtime with scoped access and rotation; never hardcode or plain-env them.
- **Audit KMS and secret access** through CloudTrail and alert on anomalies (unexpected principal, unusual volume of decrypts).
- Combine with the S3/storage encryption defaults so data is encrypted with keys you actually control.

## Pitfalls

- **Confusing "encrypted" with "protected".** If the key policy lets a compromised role decrypt, the data isn't protected from that role. Access to the key is the point.
- **One key, broad access, for everything.** Maximises blast radius — one compromise decrypts all. Separate and scope.
- **Secrets outside the secret manager.** Encrypting data at rest while credentials sit in env vars or config is a half-measure that leaks the keys to the kingdom.
- **No rotation or auditing.** A long-lived, unmonitored key is a latent risk; rotate and watch decrypt patterns.

## References

- AWS KMS best practices and Secrets Manager documentation
- Azure Key Vault / GCP KMS equivalents
- OWASP Secrets Management and Cryptographic Storage Cheat Sheets
- CWE-311 (missing encryption), CWE-522 (insufficiently protected credentials)
