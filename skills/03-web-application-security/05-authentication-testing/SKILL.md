---
format: "v2"
name: "authentication-testing"
title: "Authentication Testing"
title_fr: "Tests d'authentification"
description: "Use when testing how an app handles login, credentials, lockout, and password reset — the weaknesses that let an attacker log in as someone else — and the fixes."
description_fr: "À utiliser pour tester comment une application gère la connexion, les identifiants, le verrouillage de compte et la réinitialisation de mot de passe — les faiblesses qui permettent à un attaquant de se connecter à la place d'un autre utilisateur — et les correctifs associés."
domain: "03-web-application-security"
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

Authentication is the front door. Weaknesses here — no lockout, username enumeration, a broken reset flow — hand attackers accounts without any clever exploit. This skill covers testing the login and account-recovery surface and closing the common gaps.

### When to use it

Any app with user accounts. Test it early: auth flaws are high-impact and often shallow, so they're a fast source of real findings.

Test only accounts and apps you're authorised to. Credential attacks against systems you don't own are unauthorised access.

### Procedure

1. **Username enumeration** — does the app reveal whether an account exists? Compare responses for a valid vs invalid username on login, registration, and password reset. Different error text, different status, or different timing all leak it:
   ```
   "No such user" vs "Wrong password"   -> enumeration
   ```
2. **Brute force / lockout** — try several bad passwords for one account. Is there a lockout, rate limit, or CAPTCHA after a few failures? If not, the account is brute-forceable:
   ```
   ffuf -w passwords.txt -X POST -d 'user=admin&pass=FUZZ' -u https://app.tld/login -fr 'Invalid'
   ```
3. **Credential stuffing exposure** — no rate limiting plus no MFA means leaked-password reuse works at scale. Note whether either control exists.
4. **Password policy** — can you set `password` or `123456`? A weak policy undermines everything else.
5. **Password reset flow** — the most-abused recovery path. Check: is the reset token long, random, and single-use? Does it expire? Can you request a reset for another user and have the link leak (host-header poisoning, token in referrer)? Can you reuse an old token?
6. **Default/weak credentials** on admin panels and appliances — try the vendor defaults.
7. **Session after login** — confirm the session ID rotates at login (fixation) and that logout invalidates it (covered in the session-management skill).

### Cheatsheet

```bash
valid user   -> "incorrect password"
invalid user -> "user not found"     # <- leak (message, status, or timing)

hydra -l admin -P passwords.txt app.tld http-post-form \
  "/login:user=^USER^&pass=^PASS^:Invalid"

- token: long, random, single-use, expiring?
- reset for victim -> does link leak via Host header / referrer?
- old token still valid after use or new request?

admin:admin  admin:password  root:root  (and vendor defaults)
```

### Reading the output

- **Distinct valid/invalid responses** (text, code, or timing) = username enumeration; it turns password spraying into a targeted attack.
- **No lockout/rate limit after many failures** = brute force and credential stuffing are open. High impact combined with a weak policy.
- **A reset token that's guessable, long-lived, or reusable** = account takeover via the recovery flow — often the softest part of auth.
- **A reset link that honours an attacker-controlled Host header** = poisoned reset emails pointing at the attacker's server.
- **Default credentials working** = immediate compromise; report as critical.

### The fix

- **Uniform responses.** Same message and similar timing whether or not the account exists, on login, registration, and reset. Don't confirm account existence.
- **Rate limit and lock out.** Throttle failed logins per-account and per-IP, add CAPTCHA or exponential backoff, and alert on spikes. This blunts brute force and stuffing.
- **Strong password policy** aligned to current guidance (length over arbitrary complexity), and screen against known-breached passwords.
- **Add MFA**, especially for privileged accounts — it defeats credential stuffing even when passwords leak (see the IAM MFA skill).
- **Harden password reset:** long random single-use tokens with short expiry, invalidated after use; never build the reset URL from the Host header; don't leak the token in referrers.
- **Remove default credentials** before anything ships.

### Pitfalls

- **Enumeration through the reset/registration flow.** Teams fix the login message and forget these two leak the same info.
- **IP-only rate limiting.** Attackers rotate IPs; limit per-account too.
- **Reset tokens treated as an afterthought.** The recovery flow is often weaker than login and just as powerful. Test it as hard.
- **Client-side password policy only.** Enforce it server-side; the client check is a courtesy.

### References

- OWASP WSTG-ATHN (Authentication Testing)
- OWASP Authentication Cheat Sheet
- NIST SP 800-63B (Digital Identity — authentication)
- CWE-307, CWE-640 (weak recovery), CWE-521 (weak password requirements)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.