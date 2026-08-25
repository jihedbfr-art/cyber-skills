---
format: "v2"
name: "xss-testing"
title: "Xss Testing"
title_fr: "Tests XSS (Cross-Site Scripting)"
description: "Use when checking whether a web app reflects or stores input that executes as script in a victim's browser — covers reflected, stored, and DOM XSS plus the output-encoding fix."
description_fr: "À utiliser pour vérifier si une application web reflète ou stocke une entrée qui s'exécute comme script dans le navigateur d'une victime — couvre le XSS réfléchi, stocké et DOM, ainsi que la correction par encodage de sortie."
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

Cross-site scripting is input that ends up executing as JavaScript in someone else's browser. With it, an attacker runs in the victim's session: steal cookies, act as them, rewrite the page, pivot to internal requests. This skill covers the three flavours, how to confirm each, and the encoding discipline that stops all of them.

### When to use it

Any place user input comes back out in a page: search results, profile fields, comments, error messages that echo your input, URL fragments a script reads. If the app takes something from you and shows it to you or someone else, test it here.

### The three types

- **Reflected** — input in the request is echoed straight back in the response. Needs a crafted link the victim clicks.
- **Stored** — input is saved (a comment, a display name) and served to everyone who views it later. Worse, because it needs no bait.
- **DOM-based** — the payload never reaches the server; client-side JS reads it (from the URL, say) and writes it into the page unsafely.

### Procedure

1. Find every reflection point. Submit a unique, harmless marker like `zqx123` and search the response for it — note *where* it lands: in HTML text, an attribute, a script block, a URL.
2. Where it lands decides the payload. Start with the plain HTML-context probe and see if it comes back unencoded:
   ```
   <script>alert(document.domain)</script>
   ```
   If the angle brackets come back as `&lt;`, that context is encoded — move to the next reflection.
3. If your input lands inside an attribute, break out of it first:
   ```
   "><script>alert(document.domain)</script>
   "onmouseover="alert(document.domain)
   ```
4. If it lands inside existing JavaScript, break the string/statement instead of injecting a tag:
   ```
   ';alert(document.domain)//
   ```
5. For stored XSS, submit the payload, then load the page as a *different* user or in a clean session to confirm it fires for others, not just in your own echoed response.
6. For DOM XSS, read the client JS. Look for a source (`location.hash`, `location.search`, `document.referrer`) flowing into a sink (`innerHTML`, `document.write`, `eval`, `insertAdjacentHTML`) with no sanitisation. Trigger via the URL:
   ```
   https://app.tld/#
   ```

### Cheatsheet

```html
<!-- HTML text context -->
<script>alert(document.domain)</script>

<svg onload=alert(document.domain)>

<!-- attribute breakout -->
"><svg onload=alert(1)>
' autofocus onfocus=alert(1) x='

<!-- inside a JS string -->
';alert(1)//
</script><script>alert(1)</script>

<!-- filter-dodging shapes (use to prove encoding gaps, not to defeat a real fix) -->

<a href="javascript:alert(1)">x</a>
```

Use `document.domain` rather than `alert(1)` in reports — it proves execution *and* which origin, which matters when apps embed third-party frames.

### Reading the output

- A **popup / your JS running** is proof. But absence of a popup isn't proof of safety — the payload may execute silently or be blocked by CSP while the underlying flaw remains.
- Check whether your marker comes back **encoded** (`&lt;`, `&quot;`) or **raw**. Raw in a script or HTML context is the bug; encoded is the app doing its job.
- For stored XSS, the real confirmation is it firing in a **second, clean session** — that's the difference between "I can XSS myself" and an actual vulnerability.
- A **CSP header** may stop `alert` while the injection still lands. Note it: CSP is mitigation, not a fix, and is often bypassable.

### The fix

The root cause is output rendered without encoding for its context. Fix it at output, per context:

- **HTML text** → HTML-entity encode (`<` → `&lt;`). Every mainstream template engine does this by default; the bug is usually a "raw"/"safe" escape hatch (`|safe`, `v-html`, `dangerouslySetInnerHTML`, `innerHTML`). Hunt those.
- **HTML attribute** → attribute-encode and always quote the attribute.
- **JavaScript context** → don't inject user data into script at all; pass it as data (a JSON block the script reads), not code.
- **URL** → URL-encode, and reject `javascript:` and `data:` schemes in `href`/`src`.

Layer on top, not instead of: a strict **Content-Security-Policy** (no inline script, no `unsafe-eval`) turns many injections into non-events, and for rich text that must allow some HTML, sanitise server-side with a vetted library (DOMPurify, OWASP Java HTML Sanitizer) against an allowlist — never a homemade blocklist. Input validation helps but can't be the primary control, because the same value may be safe in one context and dangerous in another.

### Pitfalls

- **Blocklist filters give false confidence.** Stripping `<script>` does nothing against ``. Encoding at output is the fix; filtering input is not.
- **Testing only your own view.** Stored XSS that fires in your echoed response but not for other users may be reflected, not stored — confirm cross-user.
- **Missing DOM XSS because the server response is clean.** The payload never hits the server; you have to read the JavaScript.
- **Calling CSP a fix.** It reduces impact and is worth having, but the unencoded output is still the vulnerability.

### References

- OWASP WSTG-INPV-01/02 (Reflected and Stored XSS)
- OWASP XSS Prevention Cheat Sheet
- OWASP DOM-based XSS Prevention Cheat Sheet
- CWE-79

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.