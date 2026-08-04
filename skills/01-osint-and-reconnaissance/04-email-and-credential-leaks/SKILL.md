---
name: email-and-credential-leaks
domain: 01-osint-and-reconnaissance
description: Use when checking whether an organisation's email accounts and passwords have appeared in known data breaches — mapping exposure without ever using the credentials.
difficulty: beginner
tags: [osint, breaches, credentials, recon, exposure]
tools: [haveibeenpwned, dehashed]
---

## Purpose

Billions of credentials have leaked in breaches over the years, and attackers use them for credential stuffing — trying leaked passwords against your logins, betting on reuse. This skill covers checking whether an organisation's accounts appear in breach corpora, so you know your exposure before an attacker exploits it. It's passive recon against breach databases; the credentials themselves are never used against live systems.

## When to use it

External recon on an organisation, or a defensive self-assessment. High signal: a reused leaked password plus no MFA is one of the most common ways into an organisation, so knowing which accounts are exposed is directly actionable.

Check exposure — never use a found credential against a live login you're not authorised to test. Finding it is recon; using it is intrusion.

## Procedure

1. **Enumerate the org's email addresses** first (from the subdomain/OSINT work, the website, LinkedIn, or breach data itself). The email is the key you check against breach corpora.
2. **Check domains and accounts against breach data.** Have I Been Pwned tells you *which breaches* an address or domain appeared in (the domain-search feature covers a whole organisation). This gives exposure without exposing passwords:
   ```
   # HIBP: check an address, or the org's whole domain (domain search, verified owners)
   # returns: which breaches this account/domain appears in
   ```
3. **Assess the nature of each breach** — some exposed only emails, others exposed plaintext or hashed passwords. A breach that leaked passwords for your users is far more serious than one that leaked only addresses.
4. **For authorised defensive assessment**, deeper services (DeHashed and similar) can show which *credentials* are exposed — useful to gauge password-reuse risk, but handle any recovered credential as sensitive and never test it against live systems outside an authorised engagement.
5. **Map the exposure to risk**: which accounts, whether passwords leaked, and whether those accounts have MFA. Exposed credentials + no MFA + reuse is the exploitable combination.
6. **Report the exposure and the accounts affected** — the fix targets password reset, MFA, and reuse detection, not the leak itself (which you can't undo).

## Cheatsheet

```
check exposure (passive — never use the creds)
  Have I Been Pwned    account + DOMAIN search (whole-org exposure)
                       -> which breaches an address/domain appears in
  breach corpora (authorised defensive use, e.g. DeHashed) -> which credentials
                       leaked, to gauge reuse risk

what raises severity
  breach exposed PASSWORDS (plaintext/hashed) not just emails
  account has NO MFA
  password likely REUSED across services
  -> that combination = credential-stuffing takeover risk

golden rule: finding a leaked credential is recon; USING it on a live login
             you're not authorised to test is intrusion. Don't.
```

## Reading the output

- **Accounts in breaches that leaked passwords** = the actionable exposure; those users are credential-stuffing targets, especially if the password was reused. Prioritise these.
- **Whole-domain hits across many breaches** = broad organisational exposure; a pattern of staff appearing in breaches signals reuse risk across the org.
- **Breaches that leaked only email addresses** = lower direct risk (no password), but the addresses fuel phishing and enumeration.
- **An exposed privileged/admin account** = higher severity; a leaked admin credential plus reuse is a direct path in.
- **Accounts with MFA already** = the exposure is largely mitigated for those — MFA is what breaks the credential-stuffing chain even when the password leaked.

## The fix

You can't un-leak a credential, so remediation is about making the leak useless:

- **Force password resets** for exposed accounts, and screen new passwords against breach corpora (many identity providers do this) so leaked passwords can't be reused.
- **Deploy MFA**, especially for privileged accounts — it defeats credential stuffing even when the password is known (ties into the MFA skill).
- **Detect credential stuffing**: rate limiting, lockout, and alerting on distributed login attempts (the authentication-testing skill's defences).
- **Discourage password reuse** through a password manager and policy.
- **Monitor continuously** — new breaches surface over time; a one-time check goes stale. Set up breach-monitoring for your domains.

## Pitfalls

- **Using a found credential.** The line between recon and intrusion is exactly here — checking exposure is fine, logging in with a leaked password you're not authorised to use is a crime. Never cross it.
- **Treating email-only leaks as harmless.** No password leaked is lower risk, but the addresses still enable phishing and enumeration.
- **A one-time check.** Breaches keep happening; exposure is a moving target. Monitor domains continuously.
- **Fixing the password but not adding MFA.** Reset alone doesn't help if the user reuses another leaked password; MFA is what actually breaks the attack.

## References

- Have I Been Pwned (haveibeenpwned.com) — account and domain search, Pwned Passwords
- OWASP Credential Stuffing Prevention Cheat Sheet
- NIST SP 800-63B (screening against breached passwords)
- CWE-521 (weak password requirements), CWE-307 (brute force)
