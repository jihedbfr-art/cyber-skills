---
name: error-handling-and-logging
domain: 10-secure-code-review
description: Use when reviewing how code fails and what it records — catching info leaks in errors, swallowed failures that mask security events, and log injection.
difficulty: intermediate
tags: [code-review, logging, error-handling, information-disclosure]
tools: [ripgrep]
---

## Purpose

Error and logging code is where two opposite mistakes live: saying too much to the user (leaking internals) and recording too little — or the wrong thing — for defenders. Both are easy to miss because the code "works." This pass looks at the failure paths, not the happy path.

## When to use it

Any review touching exception handling, error responses, or logging. Also a good targeted sweep before a release, because leaky stack traces and swallowed exceptions accumulate quietly.

## Three things to look for

**Information disclosure on error.** A stack trace, SQL error, file path, or internal hostname returned in an HTTP response or rendered in the UI. Attackers read your errors for reconnaissance — a verbose DB error confirms the injection point, a stack trace maps your framework and versions. The user should get a generic message and a correlation id; the detail goes to the log, server-side.

**Swallowed / mislabelled failures.** `catch (e) {}` that eats an exception, a security check whose failure path silently continues, an auth error logged at `debug` and never alerted. The danger isn't the empty catch itself — it's a *security-relevant* failure that leaves no trace and stops nothing. A failed authorization that returns normally is the worst version.

**Log injection and sensitive data in logs.** Logging raw user input lets an attacker forge log lines (inject a newline + a fake entry) or break the log pipeline. And the opposite problem: passwords, tokens, full card numbers, session ids, or PII written to logs that are less protected than the database they came from. Both are common; both fail audits.

## Procedure

1. Read the catch/except blocks and the error-response construction. Does any internal detail reach the client? Is any caught exception silently dropped?
2. For each security-relevant operation (authn, authz, validation, crypto, payment), confirm its *failure* is both handled (denies) and recorded (logged at a level someone watches).
3. Scan the log statements. Are they logging untrusted input unsanitised? Are they logging secrets/PII? Both are findings.
4. Check there's a correlation id or similar so a generic user-facing error can still be traced to the detailed server log — otherwise teams "fix" leaks by logging nothing useful.

## Cheatsheet

```bash
# internal detail heading for the client
rg -n 'printStackTrace|getMessage\(\).*response|traceback\.format_exc|e\)\).*(send|render|json)'
# swallowed failures
rg -n 'catch\s*\([^)]*\)\s*\{\s*\}|except:\s*pass|except Exception:\s*pass|rescue\s*$'
# secrets / PII into logs, and raw user input into logs
rg -n 'log.*(password|token|secret|card|ssn|authorization)|log.*(request\.|getParameter|req\.body)'
```

## Reading it

- **Stack trace or DB error in the HTTP body** → information disclosure. Generic message + server-side detail is the fix.
- **Empty catch around a security check** → the check can fail open; high severity if it's authz/validation.
- **Auth failures logged at `debug` or not at all** → defenders are blind to attacks; a monitoring gap, not just a code smell.
- **Raw `request` values in a log line** → log injection risk; neutralise (strip newlines, encode) before logging.
- **Secrets/PII in logs** → the data outlives its protection. Redact at the logging boundary.

## The fix

Fail closed and quiet to the user, loud to the log. Return a generic error plus a correlation id; log the full detail server-side at a level that's actually monitored. Make sure security-relevant failures deny *and* emit an event detection can pick up (hand off to the detection-engineering and SIEM domains for what to alert on). Never log credentials or PII — redact at the sink. Sanitise user-controlled values before they enter a log line.

## Pitfalls

- **Overcorrecting into silence.** The answer to leaky errors is *move the detail to the log*, not delete it. Losing the diagnostic is its own failure.
- **Catching to hide, not to handle.** An empty catch on a security path is a fail-open.
- **Forgetting logs are a data store.** They get shipped, indexed, and shared more loosely than the DB — treat secrets/PII in them accordingly.
- **Logging attacker input verbatim.** It corrupts the very record you'll rely on during an incident.

## References

- OWASP Logging Cheat Sheet; OWASP A09:2021 Security Logging and Monitoring Failures
- CWE-209 (Info Exposure Through Error Message), CWE-532 (Info in Log), CWE-117 (Log Injection), CWE-390 (Error Not Checked)
