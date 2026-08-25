---
format: "v2"
name: "saml-security"
title: "Saml Security"
title_fr: "Sécurité SAML"
description: "Use when an app uses SAML SSO — testing for signature-wrapping, unsigned-assertion, and comment-injection attacks that forge authentication, plus correct validation."
description_fr: "À utiliser quand une application repose sur le SSO SAML : tester les attaques de signature XML wrapping (XSW), d'assertion non signée et d'injection de commentaires qui permettent de forger une authentification, et mettre en place une validation correcte."
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

SAML is the XML-based SSO protocol behind a lot of enterprise "log in with your company account". The identity provider signs an assertion vouching for the user; the app (service provider) trusts it. The attacks all target the same weak point: getting the app to accept an assertion it shouldn't — unsigned, tampered, or wrapped. This skill covers those attacks and the validation that stops them.

### When to use it

Any app using SAML for SSO (enterprise apps, federated login). SAML's XML signature model is notoriously easy to implement wrong, so it's high-value to test.

### The attack classes

- **Missing signature validation** — the app accepts an assertion that isn't signed, or doesn't check the signature at all. Forge freely.
- **XML Signature Wrapping (XSW)** — a valid signature exists, but you restructure the XML so the app validates the signature over the original element while *reading* a forged element you injected. The signature checks out; the data the app uses is yours.
- **Comment injection** — inserting an XML comment into a field (like the username) so the signature stays valid but the parser reads a different value (`admin<!---->@evil` parsed as `admin`), letting you log in as another user.
- **Assertion replay** — reusing a captured valid assertion.
- **Recipient/audience confusion** — an assertion meant for another SP accepted here.

### Procedure

1. Capture the SAML response (base64, often in a `SAMLResponse` POST parameter) and decode it to read the assertion structure.
2. **Test unsigned/removed signature** — strip the `<Signature>` element (or the assertion signature) and submit. If login still works, signatures aren't validated. Also test signing with your own key.
3. **Test signature wrapping (XSW)** — inject a forged assertion with an attacker-chosen identity while keeping the original signed element positioned so validation passes over it but the app consumes your forged one. Tooling automates the many XSW variants:
   ```
   # Burp SAML Raider extension — automate XSW permutations and re-signing tests
   ```
4. **Test comment injection** — put an XML comment in the username/NameID (`user<!---->name`) to see if the app reads a truncated/different value than what was signed.
5. **Test replay** — resubmit a captured valid assertion; a used or expired assertion should be rejected (check `NotOnOrAfter` and one-time-use).
6. **Test audience/recipient** — an assertion whose `Audience`/`Recipient` targets a different SP should be rejected here.

### Cheatsheet

```
decode:  base64 -d the SAMLResponse -> read the XML assertion

attacks to run
  strip <Signature>            -> accepted unsigned?  (no validation)
  re-sign with your own key    -> accepted?           (trusts any key)
  XSW (wrap forged assertion)  -> app reads forged, sig validates original
  comment injection in NameID  -> admin<!---->@evil parsed as admin
  replay captured assertion    -> reused / expired accepted?
  wrong Audience/Recipient     -> assertion for another SP accepted?

tool: Burp "SAML Raider" (XSW permutations, signature stripping, re-signing)
```

### Reading the output

- **Login succeeding with the signature removed or self-signed** = no/broken signature validation — total authentication bypass. Critical.
- **A successful XSW** (you become an arbitrary user while the signature "validates") = the app validates and consumes different elements; a subtle but complete forgery. Critical.
- **Comment injection changing the effective identity** = you log in as another user without breaking the signature. High impact.
- **A replayed or expired assertion accepted** = missing replay/expiry checks; captured assertions grant lasting access.
- **An assertion for another SP accepted** = missing audience/recipient validation; tokens cross trust boundaries.

### The fix

SAML validation is hard to get right by hand — use a mature, maintained SAML library and configure it strictly:

- **Validate the signature** on the assertion (not just the response), against the **expected IdP's certificate** pinned in config — reject unsigned, and reject signatures from any other key.
- **Validate what's signed is what's used** — process only the signed assertion; libraries hardened against XSW enforce this. Don't accept assertions with unexpected extra elements.
- **Canonicalise/parse safely** to defeat comment-injection (use a parser/version not vulnerable to the truncation behaviour).
- **Enforce conditions**: `NotBefore`/`NotOnOrAfter` (expiry), `Audience` == your SP, `Recipient`/`Destination` correct, and **one-time use** to block replay.
- Keep the SAML library patched — many XSW/comment bugs were library-level fixes.

### Pitfalls

- **Rolling your own SAML validation.** XML signature validation is a minefield; hand-rolled code is where XSW and stripping bugs live. Use a vetted library.
- **Validating the response signature but not the assertion.** The assertion carries the identity; if it's not the signed, consumed element, wrapping wins.
- **Not pinning the IdP certificate.** Accepting any valid signature (including the attacker's own key) defeats the point.
- **Skipping conditions.** No audience/expiry/one-time checks means replay and cross-SP reuse.

### References

- OWASP SAML Security Cheat Sheet
- Burp SAML Raider documentation
- OASIS SAML 2.0 specification (conditions, signatures)
- CWE-347 (Improper Verification of Cryptographic Signature)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.