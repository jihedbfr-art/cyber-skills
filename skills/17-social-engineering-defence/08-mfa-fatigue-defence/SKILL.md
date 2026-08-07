---
name: mfa-fatigue-defence
domain: 17-social-engineering-defence
description: Use when defending against MFA fatigue (push bombing) — where an attacker with a stolen password spams push prompts until the user approves one, and the controls that stop it.
difficulty: intermediate
tags: [social-engineering, mfa, push-bombing, fatigue, authentication]
tools: []
---

## Purpose

MFA fatigue (push bombing) is a social-engineering attack on push-based MFA: an attacker who already has the user's password repeatedly triggers login attempts, spamming the user with approval prompts until — out of annoyance, confusion, or by accident — they approve one, granting the attacker access. It's been behind major breaches precisely because it defeats MFA without any technical exploit, just persistence and human error. This skill covers defending against MFA fatigue, a specific and important gap in push-based MFA.

## When to use it

Defending push-notification MFA, and reviewing MFA implementations for this weakness. It's high-value because MFA fatigue bypasses one of the strongest common controls (MFA) through social engineering, and the defences are concrete.

## How the attack works

1. The attacker has the user's password (from phishing, a breach, or reuse — so credential controls alone aren't enough).
2. They repeatedly attempt to log in, each attempt sending a push-approval prompt to the user's phone.
3. The user, bombarded with prompts (sometimes accompanied by vishing — "this is IT, please approve" — the vishing skill), eventually approves one to make it stop, or taps approve by mistake, or is confused into thinking it's legitimate.
4. The attacker is in — MFA "worked" but was socially defeated.

## Procedure

1. **Use number matching — the primary technical defence.** Instead of a simple approve/deny push, require the user to enter or select a number displayed on the login screen. This defeats fatigue: the user can't approve a prompt without seeing the attacker's login screen (which they don't have), so blind/accidental approvals fail. Number matching is the single most effective control and should be enabled.
2. **Move toward phishing-resistant MFA.** FIDO2/WebAuthn/passkeys (the IAM MFA skill) aren't push-based at all, so they're immune to push bombing — there's no prompt to spam. For high-value accounts especially, phishing-resistant factors eliminate the whole class.
3. **Limit and rate-control prompts.** Cap the number of push prompts per time window and lock out after repeated denials, so an attacker can't bombard indefinitely. A flood of prompts should throttle and alert, not continue.
4. **Detect and alert on the pattern.** Repeated denied/ignored MFA prompts for one account is a strong attack signal — alert on it (the SOC/detection domains) and treat it as a likely credential compromise (the password is already known). The denials themselves are the detection opportunity.
5. **Train users on the specific attack.** Teach that an unexpected MFA prompt you didn't initiate should be *denied and reported*, never approved to make it stop — and that IT will never call asking you to approve a prompt (the vishing combination). A user who denies and reports an unexpected prompt stops the attack and flags the compromised password.
6. **Address the root cause too — the password is compromised.** MFA fatigue only works because the attacker has the password; strong password hygiene, breached-password screening, and phishing-resistant auth reduce how often they get there. But assume passwords will leak and defend the MFA layer.
7. **Respond to a suspected fatigue attack** — repeated prompts mean the password is known; force a password reset, investigate for compromise, and confirm no approval was granted.

## Cheatsheet

```
MFA fatigue / push bombing: attacker HAS the password -> spams push prompts -> user approves one
  (annoyance/accident/confusion) -> attacker in. defeats MFA with NO exploit, just persistence + human error.
  behind major breaches.

defend
  NUMBER MATCHING (primary): enter/select a number from the login SCREEN (attacker doesn't have it)
    -> blind/accidental approval fails. THE most effective control — enable it.
  PHISHING-RESISTANT MFA (FIDO2/WebAuthn/passkeys): not push-based -> no prompt to spam -> immune
    (best for high-value accounts — eliminates the class)
  RATE-LIMIT prompts + lockout after repeated denials (no indefinite bombardment)
  DETECT: repeated denied/ignored prompts for one account = strong attack signal -> alert + treat as
    likely credential compromise (password already known)
  TRAIN: unexpected prompt you didn't initiate -> DENY + REPORT (never approve to make it stop)
    IT never calls asking you to approve (vishing combo)
  ROOT CAUSE: password IS compromised -> pw hygiene + breached-pw screening + phishing-resistant
  respond: repeated prompts -> pw reset + investigate + confirm no approval
```

## Reading the risk

- **Push-based MFA without number matching** = vulnerable to fatigue; a simple approve/deny prompt can be spammed until the user approves. Number matching is the fix and should be enabled — its absence is the core weakness.
- **Repeated denied/ignored MFA prompts for one account** = a strong MFA-fatigue signal *and* proof the password is already compromised (the attacker has it to trigger prompts). Alert on this and force a reset — the denials are the detection opportunity.
- **A user approving a prompt to make the spam stop** = the attack succeeding through human error; training (deny and report unexpected prompts) and number matching prevent it.
- **Fatigue combined with vishing** ("this is IT, approve the prompt") = a potent combination; teach that IT never asks you to approve a prompt, and verify out-of-band (the vishing skill).
- **Phishing-resistant MFA (passkeys/FIDO2) in use** = immune to push bombing — there's no prompt to spam; the strongest defence, eliminating the class for those accounts.
- **Number matching + rate limiting + detection + phishing-resistant for high-value accounts** = MFA fatigue defended across the board.

## Pitfalls

- **Simple approve/deny push without number matching.** It's spammable until the user approves; number matching (requiring info only on the login screen) defeats blind/accidental approvals and is the primary fix.
- **Not treating repeated prompts as compromise.** A flood of MFA prompts means the attacker already has the password; force a reset and investigate, don't just wait for it to stop.
- **Training users to approve prompts to stop the spam.** The opposite of the right response; teach deny-and-report for unexpected prompts, never approve.
- **Ignoring the vishing combination.** Attackers pair fatigue with a call ("IT here, approve it"); teach that IT never asks for approval and verify out-of-band.
- **Relying on password controls alone.** MFA fatigue works because the password leaked; assume passwords will leak and defend the MFA layer (number matching, phishing-resistant, detection).
- **Not detecting the pattern.** Repeated denied prompts are a clear, alertable signal; missing it wastes an easy detection.

## References

- The IAM mfa-and-step-up (number matching, phishing-resistant) and passwordless-and-passkeys skills
- The vishing-and-smishing-awareness and IR account-compromise-response skills
- CISA guidance on MFA / number matching; notable MFA-fatigue breaches (Uber, etc.)
- FIDO Alliance phishing-resistant MFA documentation
