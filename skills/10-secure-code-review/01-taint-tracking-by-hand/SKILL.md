---
format: "v2"
name: "taint-tracking-by-hand"
title: "Taint Tracking By Hand"
title_fr: "Suivi manuel de la propagation des données (taint tracking)"
description: "Use when reviewing source for injection-class bugs — following untrusted input from where it enters (source) to where it does damage (sink) to decide if a path is exploitable."
description_fr: "À utiliser lors de la revue de code pour les failles de type injection — en suivant les données non fiables depuis leur point d'entrée (source) jusqu'à l'opération dangereuse (sink) pour déterminer si le chemin est exploitable."
domain: "10-secure-code-review"
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

Most serious code-level bugs are the same shape: untrusted data reaches a dangerous operation without being neutralised on the way. Taint tracking is the method for finding them by reading — start at a **source** (where attacker-controlled data enters), follow it to a **sink** (where it can cause harm), and check whether anything sanitises it in between. This skill is the core technique the rest of the secure-code-review domain builds on.

### When to use it

Reviewing a PR or auditing a codebase for injection-class flaws (SQLi, command injection, XSS, path traversal, SSRF, deserialization). It's how you turn "this looks risky" into "this input reaches this sink unsanitised, here's the path".

### The model

- **Source** — where untrusted data enters: request parameters, headers, cookies, uploaded files, message queues, external API responses, even the database if it holds user-influenced data.
- **Sink** — where tainted data becomes dangerous: SQL query, shell/exec call, HTML output, filesystem path, deserializer, redirect target, LDAP query.
- **Sanitiser** — what makes it safe in between: parameterisation, output encoding, allowlist validation, safe APIs. A path is exploitable when a source reaches a sink with **no effective sanitiser**.

### Procedure

1. **Find the sinks first** — there are fewer of them than sources, so it's faster to work backwards. Grep for the dangerous operations in the language you're reviewing:
   ```
   rg -n 'execute\(|exec\(|system\(|eval\(|innerHTML|readFile|deserialize|Runtime.getRuntime'
   ```
2. For each sink, ask: **what data reaches it?** Trace the argument backwards through variables and function calls to its origin.
3. Determine whether that origin is a **source** — is any part of it attacker-controlled? If it's a constant or fully internal value, move on. If it traces back to a request, it's tainted.
4. **Check the path for sanitisation.** Between source and sink, is the data parameterised, encoded for the sink's context, or validated against an allowlist? Note that the *wrong* sanitiser doesn't count — HTML-encoding data that flows into a SQL query does nothing.
5. **Confirm exploitability** before reporting. A tainted path guarded by a genuine, correct sanitiser is safe; one with none, or with a bypassable blocklist, is a finding. Write it up as the concrete path: source → (no/weak sanitiser) → sink.
6. Use a tool to widen coverage, then verify by hand — automated taint analysis finds candidates and false positives, human reading confirms which are real:
   ```
   semgrep --config p/owasp-top-ten
   ```

### Cheatsheet

```bash
SQL:     rg -n 'execute|createQuery|rawQuery|\$where'
Command: rg -n 'exec|system|popen|ProcessBuilder|Runtime'
XSS:     rg -n 'innerHTML|dangerouslySetInnerHTML|render_template_string|\|safe'
Path:    rg -n 'open\(|readFile|File\(|sendFile|include'
Deser:   rg -n 'pickle.loads|readObject|yaml.load|Marshal.load'
SSRF:    rg -n 'requests.get|urlopen|fetch\(|HttpClient|curl'

rg -n 'request\.|params|getParameter|req\.(query|body|params)|os.environ|argv'

```

### Reading the flow

- **Source → sink, nothing in between** = confirmed injection path. Report it with the exact lines.
- **A sanitiser present but wrong-context** (HTML escaping before a SQL sink, `int` cast that's later concatenated as string elsewhere) = still vulnerable. The presence of *a* sanitiser isn't safety.
- **A blocklist/regex "sanitiser"** = suspect it until proven complete; blocklists usually miss an encoding or case. Weigh it as likely-bypassable.
- **Tainted data stored then later read into a sink** = second-order injection; the source is the earlier write, easy to miss if you only look one hop back.
- **A path fully guarded by parameterisation or correct encoding** = safe; note it so you don't re-flag it.

### Making it stick (guidance for the fix)

The remediation is per-sink and lives in the specific skills (SQLi → parameterise, XSS → encode at output, etc.). The review-level habit: **neutralise at the sink, for the sink's context**, not with a generic input filter far away — because the same value can be safe in one sink and dangerous in another. Prefer safe-by-construction APIs (prepared statements, auto-escaping templates, safe deserializers) so the sanitiser can't be forgotten.

### Pitfalls

- **Starting from sources.** There are too many; you'll drown. Start from the smaller set of sinks and work back.
- **Stopping one hop back.** Data often passes through several functions and stores; follow it to the true origin, including stored/second-order paths.
- **Accepting any sanitiser as sufficient.** Wrong-context or blocklist "sanitisers" give false comfort. Check it matches the sink.
- **Trusting the SAST result wholesale.** It's a candidate list with false positives and negatives — confirm each path by reading it.

### References

- OWASP Code Review Guide
- OWASP Top 10 (Injection classes) and the per-class prevention cheat sheets
- Semgrep documentation and rule registry
- CWE-20 (Improper Input Validation), CWE-74 (Injection)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.