---
name: mfa-and-step-up
domain: 11-identity-and-access-management
description: Use when adding or reviewing multi-factor and step-up authentication — choosing factors that resist real attacks and enforcing them so they can't be bypassed.
difficulty: intermediate
tags: [iam, mfa, 2fa, step-up, phishing-resistant]
tools: [burp]
---

## Purpose

MFA is the single most effective control against credential theft — but only if the factor resists the attack you face and the enforcement can't be skipped. A poorly implemented second factor (SMS, or a check the client can bypass) gives false confidence. This skill covers choosing factors, adding step-up for sensitive actions, and testing that the enforcement actually holds.

## When to use it

Designing authentication for any app that matters, hardening after a credential-stuffing scare, or reviewing an existing MFA implementation for bypasses. Pairs with the authentication-testing and session-management skills.

## Choosing factors (weakest to strongest)

- **SMS / email OTP** — better than nothing, but phishable and vulnerable to SIM-swap. Avoid for high-value accounts; acceptable as a fallback only.
- **TOTP (authenticator app)** — a shared-secret 6-digit code. Phishable (a fake site relays it in real time) but not SIM-swappable. A reasonable default.
- **Push notification** — convenient, but vulnerable to **MFA fatigue** (bombing the user with prompts until they approve). Needs number-matching to be safe.
- **Phishing-resistant: FIDO2 / WebAuthn / passkeys** — the strong choice. The credential is bound to the origin, so a phishing site can't relay it. Use these for privileged accounts and, ideally, everyone.

## Procedure (reviewing/testing)

1. **Confirm MFA is actually enforced server-side.** The classic bypass: the app checks the password, sets a session, and treats MFA as a second page the client can skip. Try navigating straight to a post-login endpoint after the password step — if it works, MFA is cosmetic.
2. **Test the OTP flow for weaknesses**: is the code rate-limited (or can you brute-force a 6-digit code)? Does it expire? Is it single-use? Can you reuse a code, or request unlimited new ones?
3. **Test MFA fatigue resistance** on push: does approving require number-matching, or can an attacker spam prompts until the user taps approve?
4. **Test step-up** for sensitive actions (change email/password, high-value transfer) — are these gated behind a fresh factor, or does a normal session suffice? A session hijacked or a CSRF shouldn't be able to perform them without re-auth.
5. **Test recovery/backup flows** — account recovery that bypasses MFA (email a reset that skips the second factor) undermines the whole control. This is a favourite bypass.
6. **Check remember-this-device** doesn't weaken things (long-lived, transferable trust tokens).

## Cheatsheet

```
factor strength:  SMS < TOTP < push(w/ number-match) < FIDO2/WebAuthn/passkey

bypass tests
  [ ] skip the MFA step -> reach post-login endpoint directly? (cosmetic MFA)
  [ ] OTP: brute-forceable? reusable? no expiry? unlimited re-requests?
  [ ] push: MFA fatigue -> spam prompts, no number-matching?
  [ ] sensitive action performed WITHOUT step-up re-auth?
  [ ] account recovery path SKIPS MFA? (the classic backdoor)
  [ ] remember-device token long-lived / portable?

step-up = require a fresh factor at the moment of a sensitive action,
          not just at login.
```

## Reading the output

- **Reaching a logged-in state without completing MFA** = enforcement bypass; the second factor is theater. Critical — MFA that can be skipped protects no one.
- **A brute-forceable OTP** (no rate limit on a 6-digit code) = the factor is defeatable in thousands of tries; a real weakness.
- **Push with no number-matching** = MFA fatigue works; users get bombed into approving. Enforce number-matching.
- **A recovery flow that skips MFA** = the whole control has a back door; often the easiest bypass and frequently missed.
- **Sensitive actions with no step-up** = a stolen session or CSRF can do the damage MFA was meant to prevent.

## The fix

- **Enforce MFA server-side** as part of establishing the authenticated session — the session isn't fully authenticated until the second factor is verified. No client-skippable step.
- **Prefer phishing-resistant factors** (FIDO2/WebAuthn/passkeys), especially for admins; they defeat real-time phishing that TOTP and SMS don't.
- **Harden OTP**: rate-limit attempts, short expiry, single-use, and limit re-requests.
- **Number-matching on push** to kill MFA fatigue, and alert on repeated denied prompts.
- **Step-up for sensitive actions** — require a fresh factor at the moment of the action, not just at login.
- **Secure the recovery path** to the same bar as login — recovery must not be an MFA bypass.

## Pitfalls

- **Client-side MFA enforcement.** If the second factor is a page the client can skip, it's not enforced. Bind it to the session server-side.
- **SMS for high-value accounts.** SIM-swap and phishing make it weak; reserve it as a last-resort fallback.
- **Ignoring recovery flows.** The strongest MFA is undone by a reset email that skips it.
- **Push without number-matching.** Fatigue attacks work; users approve to make the prompts stop.
- **Login-only MFA.** Without step-up, a hijacked session performs sensitive actions freely.

## References

- OWASP Multifactor Authentication Cheat Sheet
- NIST SP 800-63B (authenticator assurance levels)
- FIDO Alliance / WebAuthn documentation
- CWE-287, CWE-308 (single-factor where multi-factor required)
