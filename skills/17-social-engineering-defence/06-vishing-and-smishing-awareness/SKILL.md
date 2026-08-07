---
name: vishing-and-smishing-awareness
domain: 17-social-engineering-defence
description: Use when defending against phone (vishing) and SMS (smishing) social engineering — the non-email channels attackers use to bypass email defences and pressure people directly.
difficulty: beginner
tags: [social-engineering, vishing, smishing, phone, sms]
tools: []
---

## Purpose

Not all social engineering comes by email. Vishing (voice phishing — phone calls) and smishing (SMS phishing — text messages) attack through channels your email security doesn't cover, and the phone especially adds real-time pressure a written message can't. These channels have grown as email defences improved, and they've been behind major breaches (help-desk vishing to reset MFA). This skill covers defending against phone and SMS social engineering, extending awareness beyond the inbox.

## When to use it

Building awareness and controls for the non-email social-engineering channels, which are often under-addressed because programmes focus on email phishing. Especially relevant for help desks and support staff (common vishing targets) and anyone who can be reached by phone/SMS.

## The channels and their tactics

- **Vishing (voice)** — a phone call impersonating IT support, a bank, a vendor, or an executive, using real-time conversation to pressure the target into revealing credentials, resetting MFA, or taking an action. The live interaction adds pressure and adaptability a written phish lacks — the attacker responds to hesitation in real time.
- **Smishing (SMS)** — a text with a malicious link or a request, exploiting that SMS feels informal and urgent (a "package delivery" text, a "bank alert", an "MFA code request"). Links in SMS are harder to inspect on a phone, and people trust texts more than emails.
- **A common goal: MFA and account access** — vishing the help desk to reset a target's MFA, or smishing to capture an MFA code, is a major vector for bypassing strong authentication.

## Procedure

1. **Extend awareness beyond email.** Security training often focuses on email phishing; users and especially support staff need to know that phone and SMS are attack channels too, using the same trust-and-urgency tactics. This gap is common and exploited.
2. **Harden the help desk / support process — a key control.** Help-desk vishing (calling to reset a password/MFA by impersonating an employee) is a top vector. Require strong identity verification before account actions — not knowledge an attacker can research (name, DOB, employee ID), but out-of-band or strong verification. Weak help-desk verification is how attackers bypass MFA by resetting it.
3. **Teach the core defence: verify out-of-band, resist pressure.** As with BEC, the defence is verification through a known channel — a caller claiming to be IT should be verified by calling IT back on a known number, not trusted because they called. And recognise that urgency/pressure is a manipulation tactic; legitimate requests survive "let me verify and call you back".
4. **Address MFA-specific attacks** — teach that IT will never ask for your MFA code or password, that an unexpected MFA prompt you didn't initiate should be denied and reported (ties to the MFA-fatigue skill), and that help-desk MFA resets need strong verification.
5. **Make smishing recognisable** — unexpected texts with links, urgency, or requests for codes/info; don't click SMS links, verify the sender through official channels. SMS's informality and trust are the exploit.
6. **Provide a reporting path** for suspicious calls and texts, like email phishing (the reporting-culture skill) — a vished/smished user who reports helps catch a campaign.
7. **Run awareness (and authorised vishing simulations) for high-risk roles** — help desk, finance, executives — since these channels target them specifically.

## Cheatsheet

```
not all social engineering is email. VISHING (voice) + SMISHING (SMS) bypass email defences.
  grown as email defences improved ; behind major breaches (help-desk vishing to reset MFA)

VISHING (phone): impersonate IT/bank/vendor/exec ; LIVE pressure + adaptability (responds to hesitation)
SMISHING (SMS): malicious link / request ; informal + urgent, links hard to inspect on phone, texts trusted
common goal: MFA + ACCOUNT ACCESS (vish help desk to reset MFA / smish to capture code)

defend
  EXTEND awareness beyond email (users + esp. SUPPORT STAFF)
  HARDEN HELP DESK (key): strong identity verification before account actions
    NOT researchable knowledge (name/DOB/employee ID) — weak verification = MFA-reset bypass
  CORE: verify OUT-OF-BAND (call IT back on a KNOWN number) + resist PRESSURE
    (legit requests survive "let me verify + call back")
  MFA: IT never asks for your code/password ; unexpected prompt -> deny + report
  smishing: don't click SMS links ; verify sender via official channel
  REPORTING path for calls/texts ; awareness + authorised vishing sims for high-risk roles
```

## Reading the risk

- **Help-desk process that resets passwords/MFA on weak verification** (researchable knowledge like name/DOB/employee ID) = a top vishing vector; attackers call impersonating an employee and reset their MFA to bypass strong authentication. Strong help-desk verification is the key control.
- **Awareness focused only on email** = the phone and SMS channels are under-defended; users and support staff don't recognise them as attack vectors. A common, exploited gap.
- **A caller creating urgency and pressure** = the manipulation tactic; the live interaction lets the attacker adapt to hesitation. The defence is verifying out-of-band and recognising pressure as a red flag — legitimate requests survive a callback.
- **A request for an MFA code or password** = always illegitimate (IT never asks); teach this as an absolute. Capturing MFA codes via vishing/smishing bypasses MFA.
- **Unexpected SMS with links or code requests** = smishing; SMS's informality and trust are the exploit, and links are hard to inspect on a phone. Don't click, verify the sender.
- **Hardened help desk, out-of-band verification, MFA awareness, and a reporting path across channels** = defence extended beyond email to where attackers moved.

## Pitfalls

- **Focusing awareness only on email.** Attackers moved to phone and SMS precisely because email defences improved; users and support staff need to recognise these channels. The gap is common and exploited.
- **Weak help-desk verification.** Resetting passwords/MFA on researchable knowledge lets attackers bypass MFA by vishing the help desk — a top real-world vector. Require strong verification.
- **Trusting a caller because they called.** Verify out-of-band (call back on a known number); the attacker controls the inbound call. Resist the real-time pressure.
- **Not teaching that IT never asks for MFA codes/passwords.** Users tricked into revealing codes hand over MFA; make this an absolute rule.
- **Clicking SMS links.** They're hard to inspect on a phone and SMS is trusted; don't click, verify the sender.
- **No reporting path for calls/texts.** Without one, vishing/smishing campaigns go unreported and uncaught; extend reporting beyond email.

## References

- The bec-detection, mfa-fatigue-defence, reporting-culture, and IAM mfa-and-step-up skills
- CISA and FTC guidance on vishing/smishing
- Notable breaches via help-desk vishing / MFA-reset social engineering
- NIST identity-verification guidance for help desks
