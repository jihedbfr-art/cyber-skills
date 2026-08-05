# 14 — Cryptography & PKI

You almost never need to invent crypto — you need to use the right primitive correctly and manage the keys. This domain is about that: choosing algorithms that aren't broken, using them the way they were designed, and running the certificate and key infrastructure around them.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [choosing-the-right-primitive](01-choosing-the-right-primitive/SKILL.md) | Pick hash/cipher/KDF for the job | ✅ |
| 02 | [password-hashing](02-password-hashing/SKILL.md) | Argon2/bcrypt/scrypt parameters that hold up | ✅ |
| 03 | [symmetric-encryption-correctly](03-symmetric-encryption-correctly/SKILL.md) | AEAD, nonces, and the mistakes that leak | ✅ |
| 04 | [tls-configuration](04-tls-configuration/SKILL.md) | Server-side TLS that scores clean and stays usable | ✅ |
| 05 | [certificate-management](05-certificate-management/SKILL.md) | Issue, rotate, revoke, and don't let them expire | ✅ |
| 06 | [key-management](06-key-management/SKILL.md) | Generation, storage, rotation, HSM/KMS | ✅ |
| 07 | [jwt-and-token-crypto](07-jwt-and-token-crypto/SKILL.md) | Sign and verify tokens without the footguns | ✅ |
| 08 | [secrets-at-rest](08-secrets-at-rest/SKILL.md) | Envelope encryption for stored secrets | ✅ |
| 09 | [crypto-agility](09-crypto-agility/SKILL.md) | Design so you can rotate algorithms later | ✅ |
| 10 | [post-quantum-readiness](10-post-quantum-readiness/SKILL.md) | Where PQC matters now, where it doesn't yet | ✅ |

This domain is complete (10/10). `choosing-the-right-primitive` is the anchor — most crypto bugs are a wrong choice, not a broken algorithm.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>