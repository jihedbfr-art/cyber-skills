# 14 — Cryptography & PKI

You almost never need to invent crypto — you need to use the right primitive correctly and manage the keys. This domain is about that: choosing algorithms that aren't broken, using them the way they were designed, and running the certificate and key infrastructure around them.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [choosing-the-right-primitive](01-choosing-the-right-primitive/SKILL.md) | Pick hash/cipher/KDF for the job | ✅ |
| 02 | password-hashing | Argon2/bcrypt/scrypt parameters that hold up | TODO |
| 03 | symmetric-encryption-correctly | AEAD, nonces, and the mistakes that leak | TODO |
| 04 | tls-configuration | Server-side TLS that scores clean and stays usable | TODO |
| 05 | certificate-management | Issue, rotate, revoke, and don't let them expire | TODO |
| 06 | key-management | Generation, storage, rotation, HSM/KMS | TODO |
| 07 | jwt-and-token-crypto | Sign and verify tokens without the footguns | TODO |
| 08 | secrets-at-rest | Envelope encryption for stored secrets | TODO |
| 09 | crypto-agility | Design so you can rotate algorithms later | TODO |
| 10 | post-quantum-readiness | Where PQC matters now, where it doesn't yet | TODO |

`choosing-the-right-primitive` (done) is the anchor — most crypto bugs are a wrong choice, not a broken algorithm.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>