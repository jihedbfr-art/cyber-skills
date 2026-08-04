---
name: token-lifecycle
domain: 11-identity-and-access-management
description: Use when designing how access and refresh tokens are issued, stored, refreshed, and revoked — closing the gaps that let stolen or stale tokens keep working.
difficulty: intermediate
tags: [iam, tokens, oauth2, jwt, revocation, sessions]
tools: [burp]
---

## Purpose

A token is a bearer credential — whoever holds it is treated as the user. That makes the whole lifecycle security-critical: how long it lives, where it's stored, how it's refreshed, and — the part everyone underestimates — how it's revoked. This skill covers designing a token lifecycle where theft has a short window and revocation actually works, rather than tokens that stay valid long after they should.

## When to use it

Designing or reviewing token-based auth (OAuth2/OIDC access + refresh tokens, API tokens, session tokens). It ties together threads from the OAuth2, JWT, and session skills into the end-to-end question: from issue to death, can a token be misused?

## The lifecycle stages

1. **Issuance** — what's in the token, how long it's valid, what it's scoped to.
2. **Storage** — where the client keeps it (and how exposed that is).
3. **Use** — how it's presented and validated on each request.
4. **Refresh** — how a short access token is renewed without re-login.
5. **Revocation** — how a token is killed before its natural expiry.

## Procedure

1. **Keep access tokens short-lived.** A short expiry (minutes to an hour) means a stolen access token has a small window. This is the cheapest, most effective control — it bounds the damage of theft without requiring perfect revocation.
2. **Use refresh tokens for continuity, and rotate them.** A longer-lived refresh token gets new access tokens. Rotate it on each use (issue a new refresh token, invalidate the old) and detect reuse of an old one as a theft signal — reuse means someone has a stolen copy, so revoke the whole chain.
3. **Store tokens with the right exposure in mind.** Tokens reachable by JavaScript (`localStorage`) are one XSS from theft; prefer secure, httpOnly cookies for session-like tokens, or platform secure storage on mobile. Never put a token in a URL.
4. **Design real revocation.** Self-contained JWTs are valid until they expire — you can't un-issue one — so short lifetimes plus a revocation mechanism (a checked denylist, or short-lived tokens backed by a session the server can kill) are how you actually cut access. Confirm logout, password change, and de-provisioning invalidate tokens promptly.
5. **Scope tokens narrowly.** Least-privilege scope and audience so a leaked token does the minimum, and can't be replayed against a different service.
6. **Test the gaps:** does an expired token still work (validation not checking `exp`)? Does logout leave the token usable? Does a rotated refresh token's predecessor still work? Does password change kill existing tokens?

## Cheatsheet

```
lifecycle design
  issuance   short-lived access token (mins-1h), narrow scope + audience
  storage    httpOnly secure cookie / secure platform storage
             NOT localStorage (XSS-exfiltratable), NEVER in a URL
  use        validate signature + exp + aud on every request
  refresh    longer-lived refresh token, ROTATE on each use,
             detect reuse of old token -> theft -> revoke the chain
  revocation logout / password change / de-provision -> tokens die promptly

the JWT revocation reality
  self-contained JWTs can't be un-issued -> valid until exp
  => short lifetimes + a revocation path (denylist or server-side session)

tests
  expired token accepted?  logout leaves token valid?
  old (rotated) refresh token still works?  pw-change kills tokens?
```

## Reading the design

- **Long-lived access tokens** = a stolen token works for hours or days; the single biggest lifecycle weakness. Shorten them and lean on refresh.
- **No refresh-token rotation / reuse detection** = a stolen refresh token grants indefinite access silently. Rotation plus reuse-detection turns theft into a detectable, revocable event.
- **Tokens in `localStorage`** = XSS becomes full token theft. Move session tokens to httpOnly cookies.
- **JWTs treated as revocable when they aren't** = a false sense of control; logout "works" in the UI but the token still validates. Add short lifetimes and a real revocation path.
- **Password change that doesn't invalidate tokens** = a compromised account whose owner resets the password is still accessible to the attacker via existing tokens. Kill them on credential change.

## The fix

- **Short access-token lifetimes + rotating refresh tokens** with reuse detection — the core pattern. Short windows bound theft; rotation makes stolen refresh tokens detectable and killable.
- **Store tokens out of JavaScript's reach** (httpOnly secure cookies / secure mobile storage), never in URLs.
- **Provide genuine revocation**: back tokens with a server-side session you can terminate, or maintain a checked denylist, so logout / password change / de-provisioning actually cut access. Accept that self-contained JWTs need short lifetimes precisely because they can't be un-issued.
- **Scope narrowly** (least privilege, correct audience) to limit what a leaked token can do and prevent cross-service replay.
- **Invalidate on the events that matter**: logout, password/credential change, role downgrade, suspected compromise.

## Pitfalls

- **Long-lived access tokens "for convenience".** Convenience for the attacker too — theft stays useful for a long time. Short + refresh instead.
- **Refresh tokens without rotation.** A stolen one is a permanent, silent backdoor. Rotate and detect reuse.
- **Assuming a JWT can be revoked by "logging out".** It can't be un-issued; without short lifetimes or a denylist, it keeps validating.
- **Tokens in localStorage or URLs.** XSS-exfiltratable and log-leaking respectively. Use httpOnly cookies / secure storage.
- **Not killing tokens on password change.** The reset that's supposed to lock an attacker out leaves them in via existing tokens.

## References

- OAuth 2.0 Security BCP (RFC 9700) — refresh token rotation, short-lived tokens
- OWASP JWT and Session Management Cheat Sheets
- RFC 7009 (OAuth Token Revocation)
- CWE-613 (Insufficient Session Expiration), CWE-522
