# 11 — Identity & Access Management

Auth is where most apps decide who you are and what you can touch, and it's easy to get subtly wrong. This domain covers the protocols (OAuth2, OIDC, SAML), the tokens they hand out, and the session that keeps you logged in — plus the ways each one breaks.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | oauth2-flows-and-pitfalls | Pick the right grant and avoid the classic mistakes | TODO |
| 02 | oidc-validation | Validate ID tokens properly | TODO |
| 03 | saml-security | Signature wrapping and assertion tampering | TODO |
| 04 | session-management | Cookie flags, rotation, fixation, timeout | TODO |
| 05 | mfa-and-step-up | Add a real second factor, not a checkbox | TODO |
| 06 | password-storage | Hashing, peppering, and breach resilience | TODO |
| 07 | rbac-and-abac-design | Model permissions that scale | TODO |
| 08 | sso-integration-review | Trust boundaries between IdP and app | TODO |
| 09 | token-lifecycle | Issue, refresh, revoke without gaps | TODO |
| 10 | passwordless-and-passkeys | WebAuthn done correctly | TODO |

TODO: domain scaffolded. This is your telecom/BSS strength area — good place to write from experience. Start with `oauth2-flows-and-pitfalls`.
