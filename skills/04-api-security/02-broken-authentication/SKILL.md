---
format: "v2"
name: "broken-authentication"
title: "Broken Authentication"
title_fr: "Authentification défaillante"
description: "Use when testing how an API authenticates callers — token issuance, validation, expiry, and credential endpoints — for the flaws that let an attacker forge or steal identity."
description_fr: "À utiliser pour tester la façon dont une API authentifie ses appelants — émission des jetons, validation, expiration et points d'entrée d'identifiants — afin de repérer les failles qui permettent à un attaquant de forger ou de voler une identité."
domain: "04-api-security"
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

APIs have no browser and no login page — authentication is a token the client sends on every request. When token issuance, validation, or the credential endpoints are weak, an attacker becomes another user. This skill covers testing API authentication end to end and the fixes. It's the broad companion to the JWT-specific skill.

### When to use it

Any API that authenticates callers — bearer tokens, API keys, session tokens. Test the login/token endpoints, the validation on protected routes, and the lifecycle (refresh, revoke) together.

### Procedure

1. **Map how auth works.** How is a token obtained (login, OAuth, API key), where is it sent (header, query — a query string is already a leak risk), and what validates it on each request?
2. **Credential endpoints without protection.** API login endpoints are prime brute-force / credential-stuffing targets because there's often no lockout or CAPTCHA. Test whether repeated failures are throttled:
   ```
   for p in $(cat pass.txt); do curl -s -X POST api.tld/login -d "{\"user\":\"a\",\"pass\":\"$p\"}"; done
   ```
3. **Token validation gaps.** Does the API actually verify the token, or just its presence? Test: a tampered token, an expired token (is `exp` enforced?), a token from another environment, no token at all on a "protected" route.
4. **Weak or guessable tokens / API keys.** Are tokens long and random? Are API keys treated as passwords (rate-limited, revocable) or as static secrets sprinkled in clients and logs?
5. **Token in the wrong place.** Tokens in the URL/query string leak into logs, proxies, and referrers. Flag it.
6. **Lifecycle.** Does logout/revocation actually invalidate the token server-side? Can a refresh token be reused after rotation? Does a password change invalidate existing tokens?
7. **JWT specifics** (alg confusion, none, weak secret) → hand off to the `jwt-attacks` skill if the token is a JWT.

### Cheatsheet

```
map:        obtain -> send (header? query?) -> validate (where/how?)
brute:      login endpoint throttled after N failures? lockout? captcha?
validate:   tampered / expired / foreign / missing token -> still accepted?
tokens:     long + random? revocable? not in URL? not in logs?
lifecycle:  logout revokes server-side? refresh rotates? pw-change kills tokens?

quick checks
  curl api.tld/me                         # no token -> 401 expected
  curl -H "Authorization: Bearer x" api.tld/me   # junk token -> 401 expected
  replay an expired token                  # accepted? -> exp not enforced
```

### Reading the output

- **A protected route answering with no/invalid/expired token** = broken validation; the API trusts presence over verification. Critical.
- **No throttling on the login/token endpoint** = brute force and credential stuffing are open — combine with any username enumeration for targeted attacks.
- **Tokens in the query string** = credential leakage into every log and proxy along the path.
- **Logout that doesn't invalidate server-side** = stolen tokens live until natural expiry regardless of the user "logging out".
- **A refresh token still valid after rotation** = token theft has a long tail; rotation isn't really rotating.

### The fix

- **Verify the token on every request**: signature/validity, expiry (`exp`), and that it was issued for this API (audience). Presence is not proof.
- **Protect credential endpoints** like any login: rate limiting, lockout/backoff, and MFA for sensitive access — APIs need this as much as web logins do.
- **Strong, random tokens**; treat API keys as secrets that can be scoped, rotated, and revoked, not static constants shipped in clients.
- **Never put tokens in URLs** — headers only. Keep them out of logs.
- **Short-lived access tokens + rotating refresh tokens**, with real server-side revocation on logout and on password change.
- Prefer a vetted auth library / gateway that enforces these consistently over per-endpoint checks that drift.

### Pitfalls

- **Checking token presence, not validity.** A route that accepts any bearer string is wide open. Test with junk and expired tokens.
- **Unthrottled API login.** The absence of a browser doesn't remove the need for lockout — it's easier to script, not harder.
- **API keys as permanent static secrets.** Unrevocable, unrotated keys leak and stay valid forever.
- **Forgetting revocation.** Issuing tokens well but never truly invalidating them undoes the lifecycle.

### References

- OWASP API Security Top 10 — API2:2023 Broken Authentication
- OWASP Authentication Cheat Sheet
- CWE-287, CWE-798 (hardcoded credentials)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.