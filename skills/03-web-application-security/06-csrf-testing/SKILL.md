---
format: "v2"
name: "csrf-testing"
title: "Csrf Testing"
title_fr: "Tests CSRF (Cross-Site Request Forgery)"
description: "Use when testing whether an app performs state-changing actions without verifying intent — letting a malicious page act as a logged-in victim — and how to stop it."
description_fr: "À utiliser pour vérifier si une application effectue des actions modifiant l'état sans vérifier l'intention de l'utilisateur — permettant à une page malveillante d'agir comme une victime connectée — et comment l'en empêcher."
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

Cross-Site Request Forgery makes a victim's browser send a state-changing request they didn't intend — because the browser automatically attaches their session cookie. If the server acts on the request without checking that the user meant it, an attacker's page can change the victim's password, email, or settings. This skill covers finding CSRF and the token/SameSite fixes.

### When to use it

Any authenticated, state-changing action that relies on a cookie for auth: change email/password, transfer, delete, update settings. Read-only endpoints and APIs authenticated purely by a bearer token in a header (not a cookie) are generally not CSRF-able — focus on cookie-authenticated writes.

### Procedure

1. Capture a sensitive state-changing request (e.g. change email) from an authenticated session.
2. Look for an **anti-CSRF token** in the request — a per-session/per-request unpredictable value in a body field or header. If there isn't one, that's the first sign it may be vulnerable.
3. Test whether the request succeeds **without** the token: remove it and replay. If the action still happens, there's no CSRF protection:
   ```
   curl -b "session=<victim-cookie>" -X POST https://app.tld/account/email \
     -d 'email=attacker@evil.tld'      # no token — does it work?
   ```
4. Test whether the token is actually **validated**, not just present: submit a wrong/blank/other-user's token. If any of those are accepted, the check is cosmetic.
5. Check **SameSite** on the session cookie. `SameSite=Strict`/`Lax` blocks most cross-site cookie sending; `None` (or unset on older browsers) leaves it open.
6. Build a proof-of-concept auto-submitting form to demonstrate impact cleanly (against your own test account):
   ```html
   <form action="https://app.tld/account/email" method="POST">
     <input name="email" value="attacker@evil.tld">
   </form><script>document.forms[0].submit()</script>
   ```
7. Watch for **method/CORS bypasses**: does switching `POST`→`GET` skip the check? Does a "JSON only" defence fall to a `text/plain` form submission?

### Cheatsheet

```
is it CSRF-able?
  - state-changing AND cookie-authenticated?           -> candidate
  - anti-CSRF token present?                            -> if no, likely vuln
  - request succeeds with token removed?               -> vulnerable
  - request succeeds with token wrong/blank/reused?    -> token not validated
  - SameSite=Strict/Lax on session cookie?             -> mitigates

common bypasses to test
  POST -> GET     (check skipped on GET?)
  remove Content-Type / use text/plain (defeats naive JSON-only checks)
  token tied to session? or accepted from any user?
```

### Reading the output

- **Action succeeds with the token removed** = no CSRF protection; a crafted page can trigger it. Confirmed vulnerability.
- **Action succeeds with an invalid/blank/other-user token** = the token is present but not validated — same result, common mistake.
- **`SameSite=Strict`/`Lax` set** significantly reduces risk even absent a token, but treat it as defence in depth, not a complete fix (Lax still allows some top-level GETs).
- **A GET variant that changes state and skips the check** = the write should never have been a GET; it's CSRF-able and also violates safe-method semantics.

### The fix

- **Anti-CSRF tokens** (synchronizer token pattern): a per-session, unpredictable token the server issues and *validates* on every state-changing request. Most frameworks provide this — turn it on and confirm it's actually checked.
- **`SameSite` cookies** (`Lax` as a sane default, `Strict` for sensitive apps) as a strong second layer that stops cross-site cookie attachment.
- **Only mutate on non-safe methods** (POST/PUT/DELETE), never GET — safe methods must not change state.
- For sensitive actions, **re-authenticate or confirm** (password prompt, step-up) so a forged request alone isn't enough.
- Token-in-header APIs (custom header the browser won't auto-attach cross-site) are inherently more resistant — but don't mix a cookie session with no token and assume safety.

### Pitfalls

- **Token present but unvalidated.** Shipping a token field the server ignores is a common false sense of security. Test that a wrong token is rejected.
- **Global token not bound to the user/session.** If any valid token works for any user, it's not protecting anyone.
- **Relying on `Referer` checks alone.** Easily absent or spoofable in edge cases; use tokens and SameSite.
- **Assuming JSON is safe.** A form can post `text/plain` that a lax parser accepts — confirm the content-type defence actually holds.

### References

- OWASP WSTG-SESS-05 (Testing for CSRF)
- OWASP CSRF Prevention Cheat Sheet
- CWE-352
- MDN — SameSite cookies

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.