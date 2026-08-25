---
format: "v2"
name: "jwt-attacks"
title: "Jwt Attacks"
title_fr: "Attaques sur les JWT"
description: "Use when an API authenticates with JSON Web Tokens — testing for algorithm confusion, none-alg, weak secrets, and unchecked claims, plus how to validate tokens correctly."
description_fr: "À utiliser quand une API s'authentifie via des JSON Web Tokens — pour tester la confusion d'algorithme, l'attaque alg:none, les secrets faibles et les claims non vérifiés, et pour valider correctement les jetons."
domain: "04-api-security"
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

JWTs fail in a handful of well-known ways, and every one comes down to the server trusting the token instead of verifying it. This skill covers the classic attacks — `alg:none`, RS256→HS256 confusion, brute-forcing weak HMAC secrets, and claim tampering — and the validation that makes them all fail.

### When to use it

Any API that carries auth in a JWT (bearer tokens, some session cookies). Decode one first (it's just base64) — if the app hands you a `header.payload.signature` string, this skill applies.

### Procedure

1. Decode the token and read the header and claims. The `alg` in the header and the trust claims (`sub`, `role`, `exp`, `iss`, `aud`) are your targets:
   ```
   jwt_tool <token>
   ```
2. **none algorithm:** change `alg` to `none`, drop the signature, and see if the server accepts an unsigned token. If it does, you can forge any claims:
   ```
   jwt_tool <token> -X a          # alg:none attack
   ```
3. **Weak HMAC secret:** if the token is HS256, try to crack the signing secret offline. A cracked secret lets you mint valid tokens:
   ```
   hashcat -m 16500 token.txt wordlist.txt
   ```
4. **Algorithm confusion (RS256 → HS256):** if the token is signed with RS256, the server has a *public* key. Try re-signing an HS256 token using that public key as the HMAC secret — a server that doesn't pin the algorithm will verify it:
   ```
   jwt_tool <token> -X k -pk public.pem
   ```
5. **Claim tampering:** once you can forge (via any of the above), test authorisation by changing `sub`/`user_id` (become another user) or `role`/`scope` (escalate).
6. **Validation gaps without forgery:** even with a good signature, check whether the server ignores `exp` (replay an expired token) or ignores `aud`/`iss` (a token minted for another service is accepted here).

### Cheatsheet

```bash
jwt_tool <token> -M pb            # playbook of common checks

jwt_tool <token> -X a             # alg:none
jwt_tool <token> -X k -pk pub.pem # RS256->HS256 key confusion
jwt_tool <token> -C -d words.txt  # dictionary attack on HS secret

echo '<token>' > t.txt && hashcat -m 16500 t.txt rockyou.txt

jwt_tool <token> -T
```

### Reading the output

- **A `none`-alg token accepted** = total auth bypass; you forge any identity. Critical.
- **A cracked HS256 secret** = same outcome, you can mint tokens at will. The weaker/shorter the secret, the faster it falls.
- **RS256→HS256 confusion accepted** = forgery using the public key. Critical.
- **An expired token still working** = missing `exp` validation; enables replay of stolen tokens.
- **A token for service X accepted by service Y** = missing `aud` check; tokens cross trust boundaries they shouldn't.

### The fix

Verify, don't trust — the server decides everything, the token proves nothing on its own.

- **Pin the algorithm** server-side. Accept exactly the one you issue (e.g. RS256); reject `none` and reject anything else. This kills both alg-confusion and none attacks.
- **Use strong keys.** For HMAC, a long high-entropy secret (not a dictionary word); better, use asymmetric RS256/ES256 so the verifier only holds a public key.
- **Validate every trust claim:** signature, `exp` (and `nbf`), `iss`, and `aud` against the values this service expects. A valid signature on a token meant for someone else is still invalid *here*.
- **Keep authorisation data server-side where you can.** A `role` claim in a token the client once controlled is a tempting escalation target; re-check critical permissions against your own store.
- Prefer a vetted library with these checks on by default over hand-rolled verification.

### Pitfalls

- **Accepting multiple algorithms.** The root of alg-confusion. One issued algorithm, one accepted algorithm.
- **Short or guessable HMAC secrets.** `secret`, the app name, an env default — all crack in seconds.
- **Checking the signature but not the claims.** A perfectly signed but expired or wrong-audience token should be rejected.
- **Storing sensitive authorisation solely in the token.** Fine for identity; risky as the sole source of truth for privilege.

### References

- OWASP API Security Top 10 — API2:2023 Broken Authentication
- OWASP JSON Web Token Cheat Sheet
- RFC 8725 (JWT Best Current Practices)
- CWE-347 (Improper Verification of Cryptographic Signature)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.