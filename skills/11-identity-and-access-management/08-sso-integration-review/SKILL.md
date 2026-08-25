---
format: "v2"
name: "sso-integration-review"
title: "Sso Integration Review"
title_fr: "Audit d'intégration SSO"
description: "Use when reviewing a single sign-on integration between an app and an identity provider — the trust-boundary mistakes that let an attacker impersonate users or bypass the IdP."
description_fr: "À utiliser pour auditer une intégration de single sign-on entre une application et un fournisseur d'identité : les erreurs de frontière de confiance qui permettent à un attaquant d'usurper des utilisateurs ou de contourner l'IdP."
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

Single sign-on delegates authentication to an identity provider, so the whole security of "who is this user" rests on the trust boundary between the app (service provider) and the IdP. Get that boundary wrong and an attacker impersonates any user or skips the IdP entirely. This skill is the integration-level review — how the app and IdP are wired together — sitting above the protocol-specific SAML and OIDC skills.

### When to use it

Reviewing any SSO setup: an app integrated with an enterprise IdP, a B2B federation, "log in with" flows. It focuses on the connective tissue and configuration rather than the protocol internals (which the SAML and OIDC validation skills cover in depth).

### Procedure

1. **Map the trust relationship.** Which IdP does the app trust, how is that trust configured (metadata, certificates, client secrets), and what exactly does the app accept as proof of identity? A fuzzy answer here is where the bugs hide.
2. **Verify the token/assertion validation** delegates correctly to the protocol checks — signature against the *right* IdP key, issuer, audience, expiry, replay protection. This is where SSO breaks; hand off to the SAML or OIDC skill for the specifics, but confirm the integration actually performs them.
3. **Check for local-auth bypass.** Does the app still have a password login or an API path that skips SSO? An SSO-protected app with a forgotten local login or a direct session endpoint undermines the whole federation. Test whether you can reach an authenticated state without going through the IdP.
4. **Check identity mapping.** How does the app map the IdP's assertion to a local account — by immutable ID (good) or by email (risky, if email can be changed/spoofed at the IdP or another IdP)? A mutable mapping key enables account takeover across identity sources.
5. **Check just-in-time provisioning and authorization.** If the app auto-creates accounts from SSO, what roles/permissions does a new user get, and can the IdP assertion influence them (a claim the app trusts for authorization)? An attacker who controls a claim shouldn't be able to self-grant privilege.
6. **Check IdP-initiated flows and multi-IdP setups.** IdP-initiated SSO (no request from the app) is riskier — it's more susceptible to replay and to accepting unsolicited assertions. In multi-IdP setups, confirm one IdP can't assert identities belonging to another's namespace.
7. **Check session and logout.** Does app logout/IdP logout actually terminate the session, and does a de-provisioned IdP user lose access promptly?

### Cheatsheet

```
review the trust boundary
  [ ] which IdP is trusted, pinned how (cert/metadata/client secret)?
  [ ] token/assertion fully validated? (-> SAML / OIDC skills)
        signature vs RIGHT key, issuer, audience, expiry, replay
  [ ] LOCAL-AUTH BYPASS: leftover password login / direct session endpoint?
  [ ] identity mapped by IMMUTABLE id, not mutable email?
  [ ] JIT provisioning: what perms? can a claim self-grant privilege?
  [ ] IdP-initiated flow accepted? (riskier — unsolicited assertions)
  [ ] multi-IdP: can IdP-A assert IdP-B's users?
  [ ] logout terminates session; de-provisioned users lose access

highest-impact bugs
  - accepting assertions from the wrong/any key -> impersonate anyone
  - local login bypassing SSO -> the federation is moot
  - email-based mapping -> cross-IdP account takeover
```

### Reading the review

- **A leftover local login or a session endpoint that skips SSO** = the whole SSO investment is bypassable; often the easiest and highest-impact finding. Close every path that doesn't go through the IdP.
- **Identity mapped by email** = an attacker who can set that email at any trusted identity source takes over the matching account. Map by immutable subject ID.
- **The app accepting assertions without pinning the IdP's key** (or accepting any valid signature) = impersonation of any user — the catastrophic SSO failure. Confirm the protocol validation pins the right key.
- **A claim in the assertion driving authorization** that the user could influence = privilege self-grant via SSO. Authorization should come from your side, not an assertion field the source controls.
- **IdP-initiated SSO accepted** = replay-prone; if the app doesn't need it, disabling it removes a class of attack.

### The fix

- **Validate rigorously and pin the IdP.** Accept assertions/tokens only from the specific trusted IdP's pinned key/metadata, with full issuer/audience/expiry/replay checks (per the SAML/OIDC skills). Never accept "any valid signature".
- **Remove or lock down bypass paths.** No forgotten local login, no session endpoint reachable without the IdP. SSO must be the only way in.
- **Map identity by immutable ID**, not email or another mutable attribute.
- **Control provisioning and authorization on your side** — don't let an IdP claim self-assign privileged roles; derive authorization from your own store keyed on the verified identity.
- **Prefer SP-initiated flows**, and be cautious with IdP-initiated SSO and multi-IdP trust.
- **Wire up logout and de-provisioning** so leaving the IdP promptly ends app access.

### Pitfalls

- **Leftover local authentication.** The classic SSO bypass — the org thinks access requires the IdP, but a password page or API path still works.
- **Email as the identity key.** Mutable and sometimes spoofable across sources; use the immutable subject identifier.
- **Trusting IdP claims for authorization.** Identity comes from the IdP; privilege should come from you, or an attacker who controls a claim escalates.
- **Accepting unsolicited/IdP-initiated assertions loosely.** More replay surface; disable if not needed.
- **Reviewing the protocol but not the wiring.** Perfect SAML/OIDC validation is undone by a bypass path or a bad identity mapping — review the integration, not just the tokens.

### References

- OWASP SAML and OAuth/OIDC cheat sheets
- OAuth 2.0 Security BCP (RFC 9700), OpenID Connect Core
- NIST SP 800-63C (federation)
- CWE-287, CWE-306 (missing authentication for a critical function)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.