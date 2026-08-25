---
format: "v2"
name: "file-upload-vulnerabilities"
title: "File Upload Vulnerabilities"
title_fr: "Vulnérabilités liées à l'upload de fichiers"
description: "Use when an app accepts file uploads — testing whether the upload can lead to code execution, stored XSS, or overwrite, and how to build a safe upload."
description_fr: "À utiliser quand une application accepte des uploads de fichiers — pour tester si l'upload peut mener à l'exécution de code, à un XSS stocké, ou à un écrasement de fichier, et comment construire un upload sûr."
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

An upload feature lets a user put a file on your server. Get the validation wrong and that file becomes a web shell, a stored XSS payload, or an overwrite of something important. This skill covers testing the upload surface for those outcomes and the layered controls that make uploads safe.

### When to use it

Any feature that accepts files: avatars, document uploads, import tools, attachments. The worst case — uploading a script that then executes — is why this is a high-severity area worth testing thoroughly.

### Procedure

1. Understand the upload: what types are accepted, where files are stored, and — critically — **is the storage location web-accessible and does it execute code**? An upload dir that runs `.php`/`.jsp`/`.aspx` is the dangerous setup.
2. **Bypass type restrictions** to upload a server-executable file. Test, in order, whether the app checks only:
   - the extension (try `shell.php`, then `.php5`, `.phtml`, `.pHp`, double extension `shell.jpg.php`, trailing dot/space, null byte on old stacks);
   - the `Content-Type` header (set it to `image/png` while the body is a script);
   - **magic bytes** (prepend a real image header to a polyglot).
   ```
   curl -F 'file=@shell.php;type=image/png' https://app.tld/upload
   ```
3. If you get an executable file stored in a web-served path, request it and confirm code execution — the critical outcome:
   ```
   curl "https://app.tld/uploads/shell.php?cmd=id"
   ```
4. If code execution isn't possible, test lesser but real impacts:
   - **Stored XSS** via an uploaded `.html`/`.svg` served inline (SVG can carry script).
   - **Path traversal / overwrite** via a crafted filename (`../../config`) letting you write outside the intended dir.
   - **Content spoofing / malicious file hosting** on a trusted domain.
5. Test **size and resource** handling — huge files or zip bombs (decompression DoS) if the app unpacks archives.

### Cheatsheet

```
extension bypass ladder
  shell.php  ->  shell.php5 / .phtml / .phar  ->  shell.pHp
  shell.jpg.php   (double extension)
  shell.php.       (trailing dot)   shell.php%00.jpg (null byte, legacy)

content-type spoof
  filename=shell.php ; Content-Type: image/png

magic-byte / polyglot
  prepend  GIF89a;  or a PNG header, then the script payload

lesser impacts if no RCE
  .svg with <script> served inline      -> stored XSS
  ../../path in filename                 -> traversal / overwrite
  serve .html on the app's origin        -> stored XSS / phishing on trusted domain

confirm RCE:  request the uploaded file, pass a command
```

### Reading the output

- **An uploaded script executing** (your command runs, the shell responds) = remote code execution, the top-severity outcome. Report immediately.
- **A polyglot/renamed file accepted and stored in a web-executable path** is RCE-adjacent even before you trigger it — flag the combination.
- **An SVG/HTML served inline that runs script** = stored XSS scoped to the app's trusted origin — serious even without RCE.
- **A filename with `../` landing outside the upload dir** = traversal/overwrite; impact depends on what you can clobber.
- **Only client-side type checks** (JS blocks it but the request still works when sent directly) = no real validation; treat as vulnerable.

### The fix

Defence in depth — no single check is enough:

- **Store uploads outside the web root**, or in a location/bucket that never executes code, and serve them through a handler (or a separate cookieless domain) rather than by direct path.
- **Validate type by allowlist**, checking both extension and content (magic bytes / a real parse), not just the `Content-Type` header. Reject anything not on the allowlist.
- **Generate the stored filename yourself** (random name + validated extension); never trust the client filename, which kills traversal and overwrite.
- **Serve with `Content-Disposition: attachment`** and a correct `Content-Type` so files download rather than render, and set `X-Content-Type-Options: nosniff`.
- **Limit size**, scan with AV where relevant, and guard archive extraction against zip bombs.
- Re-encode images server-side where possible — it strips embedded payloads.

### Pitfalls

- **Checking `Content-Type` or extension alone.** Both are attacker-controlled; validate the actual content and use an allowlist.
- **Client-side validation only.** The JS check is bypassed by sending the request directly. Enforce on the server.
- **Storing under the web root in an executable path.** The single most dangerous choice — it turns a weak filter into RCE.
- **Trusting the client filename.** That's how traversal and overwrite happen. Rename server-side.
- **Forgetting SVG/HTML.** They're "images"/"documents" that can carry script when served inline.

### References

- OWASP WSTG-BUSL-09 (Test Upload of Malicious Files)
- OWASP File Upload Cheat Sheet
- CWE-434 (Unrestricted Upload of File with Dangerous Type)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.