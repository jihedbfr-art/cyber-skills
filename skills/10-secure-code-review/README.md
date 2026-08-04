# 10 — Secure Code Review

Tools find patterns; people find bugs. This domain is about reading source the way an attacker reads it — following untrusted input from where it enters to where it does damage — and knowing which patterns are worth stopping on.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [taint-tracking-by-hand](01-taint-tracking-by-hand/SKILL.md) | Follow user input from source to sink | ✅ |
| 02 | [injection-patterns](02-injection-patterns/SKILL.md) | Spot the shapes that become SQLi/command/LDAP injection | ✅ |
| 03 | [auth-and-authz-review](03-auth-and-authz-review/SKILL.md) | Read access-control logic for gaps | ✅ |
| 04 | [crypto-misuse-review](04-crypto-misuse-review/SKILL.md) | Find weak, misused, or hand-rolled crypto | ✅ |
| 05 | [deserialization-review](05-deserialization-review/SKILL.md) | Unsafe deserialisation across languages | ✅ |
| 06 | [secrets-in-code](06-secrets-in-code/SKILL.md) | Hardcoded keys and how to grep them out | ✅ |
| 07 | [race-conditions](07-race-conditions/SKILL.md) | TOCTOU and concurrency bugs with security impact | ✅ |
| 08 | [error-handling-and-logging](08-error-handling-and-logging/SKILL.md) | Leaks, swallowed failures, log injection | ✅ |
| 09 | [dependency-and-config-review](09-dependency-and-config-review/SKILL.md) | The insecure default nobody changed | ✅ |
| 10 | [reviewing-a-pr-for-security](10-reviewing-a-pr-for-security/SKILL.md) | A repeatable checklist for day-to-day PRs | ✅ |

`taint-tracking-by-hand` is the skill the rest depend on — start there. `reviewing-a-pr-for-security` ties them together into a day-to-day pass.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>