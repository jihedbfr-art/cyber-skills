---
name: post-quantum-readiness
domain: 14-cryptography-and-pki
description: Use when assessing where post-quantum cryptography matters now versus later — the "harvest now, decrypt later" risk, and the pragmatic first steps toward quantum-resistant crypto.
difficulty: advanced
tags: [crypto, post-quantum, pqc, migration, future-proofing]
tools: []
---

## Purpose

A sufficiently large quantum computer would break the public-key cryptography (RSA, ECC) that secures most of today's key exchange and signatures. That machine doesn't exist yet — but the risk isn't purely future, because encrypted data captured today can be stored and decrypted once it does ("harvest now, decrypt later"). This skill covers thinking clearly about post-quantum cryptography: where it matters now, where it's premature, and the pragmatic steps to prepare without overreacting.

## When to use it

Assessing long-term cryptographic risk, especially for data with a long confidentiality lifetime, or planning a crypto roadmap. The goal is a measured view — neither ignoring the transition nor panicking about a timeline nobody can pin down.

## The threat, clearly

- **What quantum breaks:** asymmetric crypto — RSA and elliptic-curve (key exchange and signatures) via Shor's algorithm. This is the serious part.
- **What it mostly doesn't:** symmetric crypto (AES) and hashes are far less affected — Grover's algorithm roughly halves the effective security, so AES-256 stays strong and the response is "use adequate key sizes", not "replace it".
- **Harvest now, decrypt later:** an adversary can record today's encrypted traffic and decrypt it later once quantum is viable. So data that must stay confidential for many years is *already* at risk today, even though the quantum computer isn't here — this is the reason PQC is a present concern for some data, not purely future.

## Procedure

1. **Assess by confidentiality lifetime — the key discriminator.** Data that must stay secret for 10–20+ years (state secrets, long-term personal/health/financial records, long-lived signing keys) is exposed to harvest-now-decrypt-later and warrants attention now. Data that's only sensitive briefly (a short-lived session) has little quantum urgency.
2. **Inventory your public-key usage** — where you rely on RSA/ECC for key exchange and signatures. That inventory is what a future migration will target, and building it now (crypto-agility skill) is a concrete, useful step regardless of timeline.
3. **Prioritise crypto-agility over rushing to deploy PQC everywhere.** For most organisations, the highest-value step today is being *able* to migrate (parameterised algorithms, versioned data — the crypto-agility skill), so that when standards and libraries mature you can move quickly. Agility is the pragmatic first move.
4. **Adopt standardised PQC where it's warranted and available.** NIST has standardised post-quantum algorithms (e.g. ML-KEM/Kyber for key encapsulation, ML-DSA/Dilithium for signatures). For high-value, long-lived data, hybrid schemes (classical + PQC together) are the emerging safe approach — you keep classical security while adding quantum resistance, so a flaw in the new algorithm doesn't leave you worse off.
5. **Keep symmetric crypto in perspective** — ensure adequate symmetric key sizes (AES-256) and hash strengths, but don't treat symmetric as the emergency; the asymmetric transition is the real work.
6. **Track the standards, don't over-commit early.** PQC standards and implementations are maturing; deploying bleeding-edge, non-standard PQC prematurely carries its own risk. Match the urgency to your data's confidentiality lifetime.

## Cheatsheet

```
what quantum breaks
  ASYMMETRIC (RSA, ECC) — key exchange + signatures (Shor)   <- the real problem
  symmetric (AES) + hashes — mostly OK (Grover ~halves) -> use AES-256, adequate sizes

why it's a PRESENT concern for some data
  "harvest now, decrypt later" -> today's captured ciphertext decrypted in future
  -> data needing 10-20+ yr confidentiality is exposed NOW

prioritise by CONFIDENTIALITY LIFETIME
  long-lived secrets (records, long-term keys) -> attention now
  short-lived (session data)                   -> little urgency

pragmatic steps (in order)
  1. inventory RSA/ECC usage (what a migration targets)
  2. build CRYPTO-AGILITY (be ABLE to migrate) — highest-value step today
  3. for high-value long-lived data: adopt standardised PQC, prefer HYBRID
     (NIST: ML-KEM/Kyber KEM, ML-DSA/Dilithium signatures)
  4. track standards; don't over-deploy bleeding-edge prematurely
```

## Reading the situation

- **Long-confidentiality-lifetime data protected only by RSA/ECC** = exposed to harvest-now-decrypt-later; the case where PQC attention is warranted *now*, not later. Prioritise these.
- **No inventory of asymmetric-crypto usage** = you can't plan a migration; building it is a concrete, timeline-independent step worth doing now.
- **A system with no crypto-agility** = a future PQC migration will be a painful rewrite; agility is the pragmatic preparation regardless of when quantum arrives.
- **Panic-deploying non-standard PQC everywhere** = premature; immature implementations carry their own risk, and short-lived data doesn't need it. Match urgency to confidentiality lifetime.
- **Treating symmetric crypto as the emergency** = misplaced; AES-256 stays strong. The asymmetric transition is the work.
- **An inventory + crypto-agility + hybrid PQC for long-lived high-value data** = the measured, correct posture.

## The fix / best practice

- **Assess by confidentiality lifetime** — focus PQC attention on data that must stay secret for many years (harvest-now-decrypt-later exposure).
- **Inventory RSA/ECC usage** now — a useful, timeline-independent preparation.
- **Invest in crypto-agility** as the highest-value present step, so migration is fast when warranted (crypto-agility skill).
- **Adopt standardised PQC (NIST ML-KEM/ML-DSA), preferably hybrid**, for high-value long-lived data — hybrid keeps classical security as a safety net.
- **Ensure symmetric key sizes are adequate** (AES-256) but keep symmetric in perspective — not the emergency.
- **Track the maturing standards** and match deployment urgency to your actual risk, avoiding premature bleeding-edge commitments.

## Pitfalls

- **Ignoring harvest-now-decrypt-later.** "Quantum isn't here yet" misses that long-lived data captured today is already at risk. For long-confidentiality data, the concern is present.
- **Panicking and over-deploying immature PQC.** Rushing non-standard implementations everywhere adds risk, and short-lived data doesn't need it. Prioritise by confidentiality lifetime.
- **Skipping crypto-agility.** Without it, any PQC migration is a rewrite; agility is the pragmatic, no-regrets step.
- **Treating symmetric as the crisis.** AES-256 is fine; the asymmetric (RSA/ECC) transition is where the real work is.
- **Non-hybrid early adoption of new PQC.** A flaw in a young algorithm could leave you worse off; hybrid keeps classical protection alongside.

## References

- NIST Post-Quantum Cryptography standards (ML-KEM/FIPS 203, ML-DSA/FIPS 204)
- NIST and NCSC guidance on PQC migration and harvest-now-decrypt-later
- The crypto-agility and key-management skills (agility is the enabling prerequisite)
- NSA CNSA 2.0 suite (quantum-resistant algorithm guidance)
