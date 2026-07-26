# 03 — Web Application Security

The biggest domain in the repo, because it's where most real bugs live. Injection, broken auth, access control, the whole OWASP crowd. Each skill is written from the tester's chair and closes with the server-side fix — client-side checks stop honest users, not attackers.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [sql-injection-testing](01-sql-injection-testing/SKILL.md) | Probe and confirm SQLi, then parametrise it away | ✅ |
| 02 | [xss-testing](02-xss-testing/SKILL.md) | Reflected, stored, DOM XSS and the encoding fix | ✅ |
| 03 | [ssrf-testing](03-ssrf-testing/SKILL.md) | Reach internal services through the app | ✅ |
| 04 | [idor-and-broken-access-control](04-idor-and-broken-access-control/SKILL.md) | Object-level authorisation failures | ✅ |
| 05 | [authentication-testing](05-authentication-testing/SKILL.md) | Credential handling, lockout, session fixation | ✅ |
| 06 | [csrf-testing](06-csrf-testing/SKILL.md) | State-changing requests without intent | ✅ |
| 07 | [file-upload-vulnerabilities](07-file-upload-vulnerabilities/SKILL.md) | From upload to code execution, and the controls | ✅ |
| 08 | [xxe-injection](08-xxe-injection/SKILL.md) | XML parsers that fetch what they shouldn't | ✅ |
| 09 | [ssti-testing](09-ssti-testing/SKILL.md) | Server-side template injection to RCE | ✅ |
| 10 | [security-headers](10-security-headers/SKILL.md) | CSP, HSTS and the rest, done right | ✅ |

This domain is complete (10/10). If you're new here, do `sql-injection-testing` then `xss-testing` — they teach the two habits (never trust input, always encode output) that the rest of the domain builds on.
