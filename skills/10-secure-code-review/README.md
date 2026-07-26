# 10 — Secure Code Review

Tools find patterns; people find bugs. This domain is about reading source the way an attacker reads it — following untrusted input from where it enters to where it does damage — and knowing which patterns are worth stopping on.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [taint-tracking-by-hand](01-taint-tracking-by-hand/SKILL.md) | Follow user input from source to sink | ✅ |
| 02 | injection-patterns | Spot the shapes that become SQLi/command/LDAP injection | TODO |
| 03 | auth-and-authz-review | Read access-control logic for gaps | TODO |
| 04 | crypto-misuse-review | Find weak, misused, or hand-rolled crypto | TODO |
| 05 | deserialization-review | Unsafe deserialisation across languages | TODO |
| 06 | secrets-in-code | Hardcoded keys and how to grep them out | TODO |
| 07 | race-conditions | TOCTOU and concurrency bugs with security impact | TODO |
| 08 | error-handling-and-logging | Leaks, swallowed failures, log injection | TODO |
| 09 | dependency-and-config-review | The insecure default nobody changed | TODO |
| 10 | reviewing-a-pr-for-security | A repeatable checklist for day-to-day PRs | TODO |

`taint-tracking-by-hand` (done) is the skill the rest depend on.
