---
format: "v2"
name: "passwordless-and-passkeys"
title: "Passwordless And Passkeys"
title_fr: "Authentification sans mot de passe et passkeys"
description: "Use when implementing passwordless authentication with WebAuthn/passkeys — the phishing-resistant model, how it works, and the implementation details that make or break it."
description_fr: "À utiliser pour mettre en place une authentification sans mot de passe avec WebAuthn/passkeys : le modèle résistant au phishing, son fonctionnement, et les détails d'implémentation qui font toute la différence."
domain: "11-identity-and-access-management"
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

Passwords are the root of most authentication pain — phishing, reuse, credential stuffing, weak storage. Passkeys (built on WebAuthn/FIDO2) remove the shared secret entirely: authentication uses a private key bound to the device and to the site's origin, so there's nothing to phish or steal from a breached database. This skill covers how passkeys work, why they're phishing-resistant, and the implementation details that decide whether you get that benefit.

### When to use it

Designing new authentication, or adding a phishing-resistant option to an existing app. Passkeys are the strongest widely-available factor and increasingly the recommended default; this skill covers doing them right, complementing the MFA skill.

### How it works (and why it resists phishing)

- On registration, the authenticator (phone, laptop, security key) generates a **key pair per site**. The private key never leaves the device; the server stores only the public key.
- On login, the server sends a challenge; the authenticator signs it with the private key after a local user gesture (biometric/PIN). The server verifies with the public key.
- **The credential is bound to the site's origin.** The browser only offers the passkey to the exact origin it was registered for — so a phishing site at a look-alike domain can't invoke it. This origin binding is the property that defeats phishing, and it's why passkeys beat TOTP/SMS (which a fake site can relay in real time).
- **Passkeys** are WebAuthn credentials that sync across a user's devices (via a platform/cloud keychain), solving the recovery/portability problem that held back earlier FIDO2 security keys.

### Procedure (implementing)

1. **Implement the WebAuthn ceremonies** for registration (create a credential) and authentication (get an assertion), server-side. Use a maintained WebAuthn library rather than hand-rolling the cryptographic verification.
2. **Set the Relying Party ID (origin) correctly** — it's what binds credentials to your site and delivers the phishing resistance. A misconfigured RP ID (too broad, or wrong) weakens or breaks that binding. Verify it matches your actual domain scope.
3. **Verify the assertion properly** on login: the signature against the stored public key, the challenge matches the one you issued (anti-replay), the origin/RP ID is correct, and the signature counter (if present) hasn't gone backwards (clone detection).
4. **Handle registration binding** — verify the attestation only if your threat model needs it; for most consumer apps, trusting the platform authenticator is fine and demanding attestation adds friction.
5. **Design account recovery carefully** — this is the hard part of passwordless. If a user loses all devices, how do they get back in? A weak recovery path (email a magic link, security questions) reintroduces exactly the phishable weakness passkeys removed. Recovery must be as strong as the primary method, or it becomes the attack path.
6. **Plan the transition.** Most apps offer passkeys alongside passwords for a while — during that period, the password remains the weak link and phishable fallback, so protect it (MFA) and encourage passkey adoption. The full benefit comes when the password can be removed.

### Cheatsheet

```
why passkeys resist phishing
  key pair per SITE; private key stays on device; signed challenge
  browser offers the credential ONLY to the registered origin
  -> a look-alike phishing domain can't trigger it (unlike TOTP/SMS)

implementation must-checks
  [ ] use a maintained WebAuthn library (don't hand-roll verification)
  [ ] RP ID (origin) set correctly — this IS the phishing resistance
  [ ] verify: signature, challenge matches (anti-replay), origin, counter
  [ ] recovery path as STRONG as passkeys (weak recovery = the new attack)
  [ ] transition: passwords remain the phishable fallback until removed

passkey vs older FIDO2 security key: passkeys sync across devices (recovery/
portability solved) — which is what made passwordless practical at scale.
```

### Reading an implementation

- **A misconfigured or overly broad RP ID** = weakened or broken origin binding, which is the entire phishing-resistance benefit. Get this right or passkeys are just a fancier password.
- **Assertion verification skipping the challenge or origin check** = replay or cross-origin acceptance possible; the ceremony has to be validated fully, which is why a vetted library matters.
- **A weak account-recovery path** (magic-link email, security questions) = the phishable back door passkeys were meant to close. Recovery strength caps the whole scheme's strength.
- **Passwords kept as an equal fallback with no protection** = during transition the account is only as strong as its weakest login; protect the password with MFA and drive adoption.
- **Correct RP ID, full assertion verification, strong recovery** = the good state; you're actually getting the phishing resistance.

### Pitfalls

- **A weak recovery flow.** The most common way to undo passwordless — a phishable magic link or security questions become the real attack surface. Recovery must match the primary method's strength.
- **Misconfiguring the RP ID.** It's the origin binding that defeats phishing; get it wrong and you lose the main benefit silently.
- **Hand-rolling WebAuthn verification.** The ceremony has subtle checks (challenge, origin, counter); use a library that gets them right.
- **Treating the password fallback as harmless.** While it exists as an equal option, it's the phishable weak link. Protect and phase it out.
- **Assuming passkeys = MFA everywhere.** A passkey is a strong possession+gesture factor; whether it counts as multi-factor for a given policy depends on how it's used — confirm against your requirements.

### References

- W3C WebAuthn specification and FIDO2/CTAP
- FIDO Alliance passkeys documentation
- OWASP Authentication Cheat Sheet (passwordless section)
- NIST SP 800-63B (authenticator assurance)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.