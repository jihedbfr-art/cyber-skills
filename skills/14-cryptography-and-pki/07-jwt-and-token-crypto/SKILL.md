---
format: "v2"
name: "jwt-and-token-crypto"
title: "Jwt And Token Crypto"
title_fr: "Cryptographie des JWT et des jetons"
description: "Use when signing and verifying JWTs or similar tokens — choosing the algorithm, managing signing keys, and avoiding the cryptographic footguns that let tokens be forged."
description_fr: "À utiliser pour signer et vérifier des JWT ou jetons similaires — choisir l'algorithme, gérer les clés de signature et éviter les pièges cryptographiques qui permettent de forger des jetons."
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

JWTs and similar tokens carry identity and claims that servers trust, so the cryptography that protects them — the signature — is what stands between a valid session and a forged one. The JWT footguns are well-known and keep recurring: `alg:none`, algorithm confusion, weak HMAC secrets. This skill covers the cryptographic side of tokens (signing algorithm and key management) so tokens can't be forged; the API `jwt-attacks` skill covers testing them from the attacker's side, and this is the build-it-right counterpart.

### When to use it

Designing or reviewing how an application signs and verifies tokens. It focuses on the crypto choices — the surrounding validation logic (claims, expiry, audience) lives in the API jwt-attacks and IAM oidc-validation skills, which this complements.

### Procedure

1. **Choose the signing scheme deliberately, and pin it.**
   - **Asymmetric (RS256, ES256, EdDSA)** — the issuer signs with a private key; verifiers hold only the public key. Preferred when tokens are verified by parties other than the issuer (multiple services, third parties), because verifiers can't mint tokens.
   - **Symmetric (HS256)** — signed and verified with a shared secret. Fine for a single service that both issues and verifies, but every verifier that holds the secret can also forge, and the secret must be strong.
   Pin the accepted algorithm server-side; accept exactly the one you issue.
2. **Kill the algorithm footguns:**
   - **Reject `alg:none`** — never accept an unsigned token.
   - **Prevent algorithm confusion** — a verifier that accepts both RS256 and HS256 can be tricked into verifying an HS256 token using the RS256 *public* key as the HMAC secret. Pin one algorithm; don't let the token's header choose.
3. **Use strong keys.** For HMAC, a long high-entropy secret (not a word, not the app name) — weak HMAC secrets are crackable offline (the jwt-attacks skill shows this). For asymmetric, adequate key sizes (RSA-2048+, or ES256/EdDSA) and proper private-key protection (key-management skill).
4. **Manage the signing keys** like any high-value key — store the private/signing key in a KMS/HSM or secret manager, never hardcoded, and enable rotation. A leaked signing key means an attacker can mint valid tokens for anyone.
5. **Support key rotation with a key ID (`kid`)** — include a key identifier so verifiers can select the right key and you can rotate signing keys without downtime (and publish public keys via a JWKS endpoint for asymmetric schemes).
6. **Consider whether the token needs encryption too** — JWTs are signed (integrity/authenticity) but not encrypted by default; anyone can read the claims. If claims are sensitive, don't put them in a plain JWT, or use JWE — but usually the answer is to keep sensitive data out of the token.

### Cheatsheet

```
choose + PIN the algorithm
  asymmetric (RS256/ES256/EdDSA)  verifiers hold only public key -> can't forge
                                   -> use when others verify your tokens
  symmetric (HS256)               shared secret; every holder can forge; single-service
  server pins ONE accepted alg (don't let the token header choose)

footguns (recurring)
  alg:none            -> reject unsigned tokens, always
  alg confusion       -> accepting RS256+HS256 lets attacker sign HS256 with the
                         RS256 PUBLIC key -> pin one algorithm
  weak HMAC secret    -> long high-entropy secret (crackable offline otherwise)

keys
  store signing/private key in KMS/HSM/secret manager (NEVER hardcoded)
  rotate ; use `kid` for key selection ; JWKS endpoint for public keys
  leaked signing key = attacker mints valid tokens for anyone

remember: JWT is SIGNED, not ENCRYPTED -> claims are readable. Keep secrets OUT.
```

### Reading the design

- **`alg:none` accepted** = total forgery; any token is valid. Critical — reject unsigned tokens.
- **Multiple algorithms accepted (RS256 + HS256)** = algorithm-confusion forgery using the public key as HMAC secret. Pin one algorithm.
- **A weak/guessable HMAC secret** = offline-crackable, then the attacker mints tokens at will. Use a long high-entropy secret, or asymmetric signing.
- **The signing key hardcoded or poorly stored** = a leaked signing key is catastrophic (mint any token). Store it in a KMS/HSM and rotate.
- **Sensitive data in a plain JWT** = readable by anyone holding the token (it's signed, not encrypted). Keep secrets out or use JWE.
- **Pinned algorithm, strong/asymmetric keys in a KMS, `kid`+JWKS rotation, no secrets in claims** = cryptographically sound tokens.

### The fix / best practice

- **Pin the signing algorithm** and reject `alg:none` and any algorithm you don't issue — this kills the two classic forgery paths.
- **Prefer asymmetric signing** (RS256/ES256/EdDSA) when tokens are verified beyond the issuer, so verifiers can't forge; use HS256 only for single-service cases with a strong secret.
- **Use strong keys** and protect the signing key in a KMS/HSM/secret manager, never hardcoded.
- **Rotate signing keys** with `kid`/JWKS so rotation is seamless and a leaked key can be retired.
- **Keep sensitive data out of tokens** — JWTs are readable; don't rely on them for confidentiality.
- Pair with proper claim validation (API jwt-attacks / IAM oidc-validation skills).

### Pitfalls

- **Accepting `alg:none` or multiple algorithms.** The recurring JWT forgery bugs; pin one algorithm and reject unsigned.
- **Weak HMAC secrets.** Offline-crackable, then tokens are forgeable. Long high-entropy secret, or go asymmetric.
- **Poorly stored / never-rotated signing keys.** A leaked signing key mints tokens for anyone; protect it like the crown-jewel key it is and support rotation.
- **Assuming JWTs are encrypted.** They're signed, not encrypted — claims are readable by anyone with the token. Keep secrets out.
- **Only fixing the crypto, not the claims.** A perfectly-signed token that isn't validated for expiry/audience is still exploitable (see the API/IAM skills).

### References

- RFC 8725 (JWT Best Current Practices), RFC 7519 (JWT), RFC 7517 (JWK)
- OWASP JSON Web Token Cheat Sheet
- The API jwt-attacks, IAM oidc-validation, and key-management skills
- CWE-347 (improper signature verification), CWE-321 (hardcoded key)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.