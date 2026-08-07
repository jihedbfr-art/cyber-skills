---
name: mobile-auth-and-biometrics
domain: 05-mobile-security
description: Use when assessing mobile authentication and biometric (Face ID / fingerprint) implementations — the local-auth mistakes that let an attacker bypass the lock without the biometric.
difficulty: intermediate
tags: [mobile, authentication, biometrics, faceid, local-auth]
tools: [frida, objection]
---

## Purpose

Mobile apps use biometrics (Face ID, Touch ID, fingerprint) for convenient authentication — but implemented wrong, the biometric is decorative: an attacker with the device bypasses it without ever presenting a face or finger. The common mistake is treating biometric auth as a simple "did it succeed?" boolean that client-side code checks, rather than binding it to a cryptographic operation. This skill covers assessing mobile authentication and biometric implementations for these bypasses.

## When to use it

Assessing any app that uses local/biometric authentication, especially for gating sensitive functionality (banking apps, secret stores). Biometric bypass is a common, high-impact mobile finding because the implementation mistakes are subtle and frequent.

## Procedure

1. **Understand the two ways to use biometrics — one secure, one not.**
   - **Event-based (insecure):** the app calls the biometric API, gets back "success/failure", and *client-side code* decides whether to proceed. This is bypassable — hook the check to return success, and the biometric is skipped entirely (no face/finger needed). The decision is a client-side boolean an attacker controls.
   - **Cryptographic (secure):** the biometric authentication *unlocks a key* in the secure hardware (Keystore/Keychain/Secure Enclave) that's needed to decrypt data or complete an operation. Here the biometric is bound to a cryptographic result — bypassing the check doesn't produce the key, so it can't be skipped. This is the correct pattern.
2. **Test for the event-based bypass — the key test.** With dynamic instrumentation (Frida/Objection), hook the biometric success callback / the code that checks the result and force it to "success". If this bypasses authentication and grants access, the implementation is event-based and insecure:
   ```
   objection -g <pkg> explore    # biometric bypass helpers ; or Frida-hook the auth callback -> success
   ```
   If the app still can't proceed (because it needed a key the biometric didn't unlock), it's using the cryptographic pattern correctly.
3. **Check what the biometric gates.** Is it protecting genuine secrets (data encrypted with a biometric-bound key) or just a UI gate (a screen the app shows/hides)? A UI gate is bypassable; a cryptographic gate isn't.
4. **Assess the broader auth flow.** Beyond biometrics: is authentication also enforced server-side (a biometric unlocks local access, but the backend must still authenticate the session — the biometric shouldn't replace server auth)? Are tokens/credentials protected (the storage skill)? Is there a fallback (PIN) and is it secure?
5. **Check for local-auth-only trust.** An app that relies solely on local/biometric auth for security-critical access, without server-side enforcement, trusts the client — bypassable. Sensitive operations should be server-authenticated, not gated only by local biometrics.
6. **Report the bypass and its impact** — a biometric bypass on a banking or secrets app grants access to sensitive functionality without the biometric, a high-severity finding.

## Cheatsheet

```
biometrics implemented wrong = decorative (attacker with device bypasses w/o face/finger)

two patterns
  EVENT-BASED (insecure): biometric API -> "success/fail" boolean -> CLIENT-SIDE code decides
    -> hook the check -> return success -> biometric SKIPPED entirely. BYPASSABLE.
  CRYPTOGRAPHIC (secure): biometric UNLOCKS A KEY in secure hardware (Keystore/Keychain/Secure Enclave)
    needed to decrypt/complete the operation -> bypassing check doesn't produce the key. CORRECT.

KEY TEST (Frida/Objection): hook the biometric success callback -> force "success"
  bypasses auth + grants access? -> EVENT-BASED, insecure
  still can't proceed (needed a key)? -> cryptographic, correct
    objection -g <pkg> explore  (biometric bypass helpers)

also check
  what it GATES: real secrets (biometric-bound key) vs just a UI gate (bypassable)
  server-side auth still enforced? (biometric unlocks LOCAL access, not a substitute for backend auth)
  tokens/creds protected (storage skill) ; secure fallback (PIN)
  LOCAL-AUTH-ONLY trust for critical access = bypassable -> server must enforce
```

## Reading the implementation

- **Hooking the biometric callback to "success" bypassing authentication** = event-based, insecure implementation; the biometric is a client-side boolean an attacker controls, so it grants access without any biometric. The core finding, and common.
- **The app unable to proceed after a hooked "success"** (because it needed a key the biometric didn't unlock) = the cryptographic pattern done right; the biometric is bound to a key in secure hardware, so it can't be skipped. Note it as correct.
- **Biometrics gating only a UI screen** (not protecting encrypted data) = a bypassable UI gate; the "protected" screen is shown/hidden by client code, not cryptographically locked.
- **Local biometric auth relied on for security-critical access with no server enforcement** = trusting the client; a bypass grants access the backend should have independently gated. Sensitive operations need server-side auth.
- **A high-value app (banking, secrets) with an event-based biometric bypass** = high-severity; access to sensitive functionality without the biometric.
- **Cryptographically-bound biometrics unlocking a hardware key, with server-side auth for sensitive operations** = the secure state.

## The fix

- **Bind biometrics to a cryptographic operation** — use the biometric to unlock a key in the secure hardware (Android Keystore with `setUserAuthenticationRequired`, iOS Keychain with biometric access control / Secure Enclave) that's *needed* to decrypt data or complete the operation. Never a simple client-side success boolean.
- **Don't use event-based biometric checks** for anything security-relevant — they're bypassable by hooking the result.
- **Enforce sensitive operations server-side** — the biometric unlocks local access, but the backend must independently authenticate; local auth isn't a substitute for server auth.
- **Protect the fallback** (PIN/password) and the credentials/tokens the biometric gates (the storage skill).
- **Assume the client is compromised** — biometrics improve UX and local protection but can't be the sole gate for critical access.

## Pitfalls

- **Event-based biometric checks.** A client-side "success" boolean is bypassable by hooking; the biometric is decorative. Bind it to a cryptographic key operation instead.
- **Gating a UI screen, not data.** Showing/hiding a screen based on biometric success is bypassable; cryptographically lock the actual data.
- **Trusting local biometric auth alone for critical access.** It's bypassable with device control; sensitive operations must be server-authenticated. Biometrics complement, not replace, server auth.
- **Insecure fallback.** A weak PIN fallback undermines a strong biometric; secure the whole flow.
- **Treating the biometric as identity for the backend.** It authenticates locally to unlock access/keys; the server still needs its own authentication.

## References

- OWASP MASTG (authentication, biometric testing) and MASVS
- Android BiometricPrompt / Keystore (`setUserAuthenticationRequired`) and iOS LocalAuthentication / Keychain access control documentation
- The insecure-data-storage, dynamic-instrumentation-frida, and mobile-api-traffic skills
- OWASP mobile authentication guidance
