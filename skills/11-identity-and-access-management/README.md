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
| 07 | [rbac-and-abac-design](07-rbac-and-abac-design/SKILL.md) | Model permissions that scale | ✅ |
| 08 | [sso-integration-review](08-sso-integration-review/SKILL.md) | Trust boundaries between IdP and app | ✅ |
| 09 | [token-lifecycle](09-token-lifecycle/SKILL.md) | Issue, refresh, revoke without gaps | ✅ |
| 10 | [passwordless-and-passkeys](10-passwordless-and-passkeys/SKILL.md) | WebAuthn done correctly | ✅ |

This domain is complete (10/10). Start with `oauth2-flows-and-pitfalls`.
