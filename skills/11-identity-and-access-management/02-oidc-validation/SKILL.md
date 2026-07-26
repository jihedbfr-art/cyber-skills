---
name: oidc-validation
domain: 11-identity-and-access-management
description: Use when an app uses OpenID Connect for login — validating ID tokens correctly so an attacker can't forge or replay an identity, and the checks that are commonly skipped.
difficulty: intermediate
tags: [iam, oidc, id-token, authentication, jwt]
tools: [burp, jwt_tool]
---

## Purpose

OpenID Connect adds authentication on top of OAuth2, and the ID token is its proof of who logged in. That proof only holds if the app validates the token properly — the wrong checks skipped, and an attacker forges or reuses an identity. This skill covers the ID-token validation an OIDC client must do, and the gaps that turn "log in with X" into account takeover.

## When to use it

Any app that logs users in via OIDC — "Sign in with Google/Microsoft/Okta", or a custom identity provider. It complements the OAuth2 skill (which covers the flow) by focusing on what the client must verify about the token it gets back.

## The checks an OIDC client must perform

Validate all of these on the ID token — skipping any one is a real weakness:

1. **Signature** — verify against the provider's published keys (JWKS). An unverified token is just attacker-supplied JSON. (The JWT-attacks skill covers alg-confusion/none here — pin the algorithm.)
2. **`iss` (issuer)** — must exactly match the expected identity provider. A token from a different issuer is not your IdP.
3. **`aud` (audience)** — must match *your* client ID. A token minted for another app/client must be rejected, even if signed by the same provider.
4. **`exp` / `iat`** — the token must be unexpired; reject stale tokens.
5. **`nonce`** — the value you sent in the auth request must come back in the token, tying it to this login and preventing replay/injection of a token from elsewhere.

## Procedure

1. Capture the ID token returned after login (a JWT) and decode it.
2. **Test signature enforcement** — tamper with a claim, or try `alg: none`. If a modified/unsigned token is accepted, signature validation is broken (hand off to jwt-attacks).
3. **Test `aud` enforcement** — present a validly-signed token whose `aud` is a *different* client. If the app accepts it, an attacker can reuse a token obtained for another app to log in here. This is a frequently-missed, high-impact check.
4. **Test `iss` enforcement** — a token from an unexpected issuer should be rejected.
5. **Test `nonce` handling** — replay a token without the matching nonce, or reuse a prior token; it should fail.
6. **Test expiry** — replay an expired token; a valid-signature-but-expired token must be rejected.
7. Confirm the app derives identity from the validated token claims (`sub`), not from an untrusted field the client could influence.

## Cheatsheet

```
ID token validation checklist (all required)
  [ ] signature verified against provider JWKS; algorithm pinned (no none)
  [ ] iss  == expected identity provider (exact)
  [ ] aud  == your client_id  (reject tokens minted for other clients!)
  [ ] exp  in the future; iat sane
  [ ] nonce matches the one sent in the auth request
  [ ] identity taken from validated `sub`, not a client-supplied field

high-value tests
  - token with aud = another client  -> accepted? (token reuse / takeover)
  - alg:none / tampered claim         -> accepted? (broken signature check)
  - expired token replayed            -> accepted? (no exp check)
```

## Reading the output

- **A token with a different `aud` accepted** = token reuse across applications; an attacker who gets a token for any app using the same IdP logs in as that user here. High impact and commonly missed.
- **A tampered or `alg:none` token accepted** = broken signature validation; identities can be forged outright. Critical.
- **Wrong `iss` accepted** = the app trusts tokens from issuers it shouldn't.
- **Missing `nonce` check** = token replay/injection is possible; the login isn't bound to this session.
- **Expired token accepted** = stolen tokens have no time limit.
- **All checks enforced** = the client is validating correctly; note it as the good state.

## The fix

- **Use a certified OIDC library** and enable full validation — don't hand-roll token parsing. The library validates signature, `iss`, `aud`, `exp`, and `nonce` when configured correctly; your job is to configure the expected issuer, audience, and JWKS.
- **Pin the signing algorithm** and fetch keys from the provider's JWKS endpoint (with rotation), never trust an alg from the token header.
- **Always check `aud` against your client ID** — this is the one teams skip, and it's what stops cross-app token reuse.
- **Send and verify a `nonce`** per login.
- **Derive the user from `sub`** (stable subject identifier) on the validated token, not from email or a mutable claim, and not from anything the client supplied.

## Pitfalls

- **Skipping the `aud` check.** The classic OIDC bug — a signed token for another client sails through and logs the attacker in.
- **Verifying signature but not claims.** A perfectly signed token for the wrong audience/issuer, or expired, is still invalid *here*.
- **Trusting the token header's `alg`.** Opens alg-confusion/none. Pin it.
- **Using `email` as the identity key.** Emails change and can be spoofed across providers; use the immutable `sub`.

## References

- OpenID Connect Core — ID Token Validation section
- OAuth 2.0 Security BCP (RFC 9700)
- OWASP JSON Web Token Cheat Sheet
- CWE-287, CWE-347
