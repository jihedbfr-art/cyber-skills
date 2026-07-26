# 11 — Identity & Access Management

Auth is where most apps decide who you are and what you can touch, and it's easy to get subtly wrong. This domain covers the protocols (OAuth2, OIDC, SAML), the tokens they hand out, and the session that keeps you logged in — plus the ways each one breaks.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [oauth2-flows-and-pitfalls](01-oauth2-flows-and-pitfalls/SKILL.md) | Pick the right grant and avoid the classic mistakes | ✅ |
| 02 | [oidc-validation](02-oidc-validation/SKILL.md) | Validate ID tokens properly | ✅ |
| 03 | [saml-security](03-saml-security/SKILL.md) | Signature wrapping and assertion tampering | ✅ |
| 04 | [session-management](04-session-management/SKILL.md) | Cookie flags, rotation, fixation, timeout | ✅ |
| 05 | [mfa-and-step-up](05-mfa-and-step-up/SKILL.md) | Add a real second factor, not a checkbox | ✅ |
| 06 | [password-storage](06-password-storage/SKILL.md) | Hashing, peppering, and breach resilience | ✅ |
| 07 | rbac-and-abac-design | Model permissions that scale | TODO |
| 08 | sso-integration-review | Trust boundaries between IdP and app | TODO |
| 09 | token-lifecycle | Issue, refresh, revoke without gaps | TODO |
| 10 | passwordless-and-passkeys | WebAuthn done correctly | TODO |

TODO: domain scaffolded. This is your telecom/BSS strength area — good place to write from experience. Start with `oauth2-flows-and-pitfalls`.
