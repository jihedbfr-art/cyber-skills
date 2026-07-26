# 04 — API Security

APIs fail differently from the web apps that front them. No browser to enforce anything, authorisation checked per-endpoint (or forgotten), and object IDs handed straight to the client. This domain follows the OWASP API Top 10 shape.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [broken-object-level-authorization](01-broken-object-level-authorization/SKILL.md) | Test BOLA/IDOR on API objects — the #1 API bug | ✅ |
| 02 | [broken-authentication](02-broken-authentication/SKILL.md) | Token issuance, expiry, and validation flaws | ✅ |
| 03 | [excessive-data-exposure](03-excessive-data-exposure/SKILL.md) | Endpoints that over-return and trust the client to filter | ✅ |
| 04 | [rate-limiting-and-resource-abuse](04-rate-limiting-and-resource-abuse/SKILL.md) | Missing quotas, brute force, cost attacks | ✅ |
| 05 | [mass-assignment](05-mass-assignment/SKILL.md) | Binding fields the client shouldn't control | ✅ |
| 06 | [graphql-security](06-graphql-security/SKILL.md) | Introspection, depth, batching abuse | ✅ |
| 07 | [jwt-attacks](07-jwt-attacks/SKILL.md) | alg confusion, weak secrets, unchecked claims | ✅ |
| 08 | api-gateway-hardening | Where to enforce auth, rate limits, schema | TODO |
| 09 | grpc-security | Auth and input validation over gRPC | TODO |
| 10 | webhook-security | Verifying and locking down inbound webhooks | TODO |

Start with `broken-object-level-authorization`. It's the flaw most APIs actually ship with.
