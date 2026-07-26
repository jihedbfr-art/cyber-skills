---
name: oauth2-flows-and-pitfalls
domain: 11-identity-and-access-management
description: Use when choosing or reviewing an OAuth2 flow — picking the right grant for the client type and avoiding the redirect, state, and PKCE mistakes that break it.
difficulty: intermediate
tags: [iam, oauth2, oidc, authorization, tokens]
tools: [burp, curl]
---

## Purpose

OAuth2 is an authorisation framework, not a drop-in login, and most of its security problems come from picking the wrong flow or skipping one of its safety parameters. This skill covers which grant fits which client, and the handful of checks that separate a safe integration from an account-takeover waiting to happen.

## When to use it

Designing an integration that delegates access, reviewing one someone else built, or testing an OAuth deployment during an assessment. If you see `response_type`, `redirect_uri`, `code`, or `state` in the traffic, this is your skill.

## Choosing the grant

- **Authorization Code + PKCE** — the default for essentially everything today: web apps, SPAs, mobile, desktop. PKCE protects the code exchange even for public clients. If you're unsure, this is the answer.
- **Client Credentials** — machine-to-machine, no user involved. A service calling another service.
- **Device Code** — inputs-constrained devices (TVs, CLI tools) where the user authorises on a second device.
- **Implicit** — deprecated. It returned tokens in the URL fragment and leaked them through history and referrers. Don't use it; migrate SPAs to Auth Code + PKCE.
- **Resource Owner Password Credentials** — also deprecated. It has the app handle the user's password, defeating the point of OAuth. Avoid.

## Procedure (reviewing/testing a flow)

1. Identify the grant from the authorization request. `response_type=code` with a `code_challenge` is Auth Code + PKCE (good); `response_type=token` is implicit (flag it).
2. **redirect_uri validation** — the classic bug. The server must match it against a pre-registered value *exactly*, not by prefix or substring. Test variations to see what's accepted:
   ```
   &redirect_uri=https://attacker.tld
   &redirect_uri=https://app.tld.attacker.tld
   &redirect_uri=https://app.tld/callback/../../evil
   &redirect_uri=https://app.tld@attacker.tld
   ```
   Any of these being accepted means the code can be sent to an attacker.
3. **state parameter** — it's the CSRF defence for the callback. Confirm it's present, unpredictable, and *validated* on return. Remove it and see if the flow still completes; if it does, the login is CSRF-able.
4. **PKCE** — for public clients, confirm `code_challenge` goes out and the matching `code_verifier` is required at token exchange. Replaying a code without the verifier should fail.
5. **Scope and consent** — check the app requests only the scopes it needs and that scope can't be silently upgraded. Test whether adding a scope to the request escalates access without fresh consent.
6. **Token handling at the client** — where do tokens land? Access tokens in `localStorage` are XSS-exfiltratable; refresh tokens exposed to JS are worse. Confirm sensible storage and short access-token lifetimes.

## Cheatsheet

```
# authorization code + PKCE request (the good shape)
GET /authorize?response_type=code
    &client_id=...&redirect_uri=https://app.tld/callback
    &scope=openid%20profile&state=<random>&code_challenge=<hash>&code_challenge_method=S256

# token exchange (verifier proves possession)
POST /token
  grant_type=authorization_code&code=...&redirect_uri=...&client_id=...&code_verifier=<original>

# things to test
redirect_uri: exact match?  open redirect on the registered host?
state: present, random, validated on callback?
code: single-use? bound to the client? expires fast?
pkce: verifier required? S256 not plain?
```

## Reading the output

- **A loosely-matched `redirect_uri` accepted** = authorization-code theft, which is account takeover. The single highest-impact OAuth bug.
- **Missing or unvalidated `state`** = login CSRF; an attacker can force a victim into the attacker's session or vice versa.
- **A code that's reusable, or accepted without PKCE verifier** = the exchange isn't bound to the legitimate client.
- **Silent scope upgrade** = the app can gain access the user never consented to.
- **Tokens in `localStorage`** = one XSS from full token theft.

## The fix

- **Exact-match redirect URIs** against a registered allowlist — full string, scheme included, no wildcards, no prefix matching. Also make sure the registered host has no open redirect, which reintroduces the problem.
- **Auth Code + PKCE everywhere**, retire implicit and password grants.
- **Mandatory, validated `state`** (or the OIDC `nonce`), unpredictable and single-use.
- **Least scope**, explicit consent, no silent escalation.
- **Short-lived access tokens**, refresh tokens stored server-side or in secure httpOnly cookies rather than JS-reachable storage; rotate refresh tokens.
- Treat OAuth as *authorisation*. For "log in with", use **OIDC** on top and validate the ID token properly — OAuth alone was never an authentication protocol, and using it as one is its own class of bug.

## Pitfalls

- **Prefix/substring redirect matching.** `https://app.tld.evil.com` passes a naive check. Exact match only.
- **Implicit flow still in the wild.** Tokens in the URL leak; migrate SPAs to PKCE.
- **`state` present but never checked.** Sending it isn't the defence — validating it on the callback is.
- **Confusing OAuth with login.** Delegated access is not identity; bolt OIDC on and verify the ID token's signature, `iss`, `aud`, `exp`, and `nonce`.

## References

- RFC 6749 (OAuth 2.0), RFC 7636 (PKCE), RFC 9700 (OAuth 2.0 Security BCP)
- OpenID Connect Core specification
- OWASP WSTG-ATHZ and OAuth cheat sheets
