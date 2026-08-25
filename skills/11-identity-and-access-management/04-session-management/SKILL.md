---
format: "v2"
name: "session-management"
title: "Session Management"
title_fr: "Gestion des sessions"
description: "Use when reviewing how an app issues, protects, and ends sessions — cookie flags, fixation, rotation, timeout — and the settings that make sessions safe."
description_fr: "À utiliser pour auditer la façon dont une application émet, protège et termine ses sessions : attributs de cookie, fixation de session, rotation, expiration, et les réglages qui rendent une session réellement sûre."
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

A session is what keeps a user logged in after they authenticate, which makes the session token as valuable as the password. This skill covers the properties a session must have — protected in transit, unpredictable, rotated at login, and actually killed at logout — and how to check each one.

### When to use it

On any app that keeps users logged in. Fast to review, and the failures (fixation, missing flags, sessions that outlive logout) are common and directly exploitable.

### Procedure

1. Capture the session cookie after login and inspect its flags. The four that matter are `Secure`, `HttpOnly`, `SameSite`, and a sane `Path`:
   ```
   curl -sI https://app.tld/login -d 'user=..&pass=..' | grep -i set-cookie
   ```
   Want: `Set-Cookie: session=...; Secure; HttpOnly; SameSite=Lax`.
2. **Session fixation** — the token must change at the moment privilege changes. Note the pre-login session ID, authenticate, and check it's a *new* ID afterwards. If it's the same, an attacker who planted it is now logged in as the victim.
3. **Predictability** — the token should be long and random. If IDs look sequential or short, that's a problem; a good token is opaque and high-entropy.
4. **Logout** — after logging out, replay a captured authenticated request with the old cookie. It must fail. A session that still works post-logout means logout is cosmetic (client-side only).
5. **Timeout** — confirm an idle session expires server-side and that there's an absolute lifetime, not just a client timer.
6. **Scope** — check the cookie isn't over-shared (`Domain` too broad, `Path=/` when it needn't be) and that it isn't reflected anywhere (URL, logs).

### Cheatsheet

```
Set-Cookie: session=<opaque-random>; Secure; HttpOnly; SameSite=Lax; Path=/

Secure      -> only sent over HTTPS
HttpOnly    -> not readable by JS (blunts XSS token theft)
SameSite    -> Lax or Strict (CSRF defence in depth)
rotation    -> new session id issued at login and privilege change
logout      -> server-side invalidation, old token rejected
timeout     -> idle + absolute expiry, enforced server-side
entropy     -> long, unpredictable, not sequential
```

### Reading the output

- **Missing `HttpOnly`** = an XSS bug becomes session theft. **Missing `Secure`** = the token can leak over HTTP.
- **Same session ID before and after login** = session fixation, straightforwardly exploitable.
- **Old cookie still authenticated after logout** = no server-side invalidation; stolen or shared tokens live forever.
- **Short/sequential tokens** = guessable sessions; test whether adjacent IDs belong to other users.
- **`SameSite=None` without good reason** widens CSRF exposure.

### The fix

- Set `Secure`, `HttpOnly`, and `SameSite=Lax` (or `Strict` for sensitive apps) on the session cookie. Keep `Domain`/`Path` as narrow as the app allows.
- **Rotate the session identifier on every login and privilege change** — this is the fixation fix. Invalidate the pre-auth session.
- Generate tokens from a CSRF-grade random source; keep them opaque and long. Let the framework's session manager do this rather than rolling your own.
- **Invalidate server-side on logout** — delete or expire the session record, don't just clear the cookie. Offer "log out everywhere" for high-value accounts.
- Enforce both idle and absolute timeouts on the server. The client timer is a courtesy, not a control.
- Never put the session token in a URL, and keep it out of logs.

### Pitfalls

- **Client-side-only logout.** Clearing the cookie in the browser while the server still honours the token is not logout.
- **Trusting `SameSite` as the whole CSRF story.** It helps, but pair it with proper anti-CSRF tokens on state-changing requests.
- **Rotating at login but not at privilege elevation.** A step-up to admin should also refresh the session.
- **Rolling your own token generator.** Use the platform's session manager; homemade entropy is where predictable IDs come from.

### References

- OWASP Session Management Cheat Sheet
- OWASP WSTG-SESS (Session Management Testing)
- CWE-384 (Session Fixation), CWE-613 (Insufficient Session Expiration)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.