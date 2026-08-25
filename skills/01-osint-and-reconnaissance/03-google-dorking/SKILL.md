---
format: "v2"
name: "google-dorking"
title: "Google Dorking"
title_fr: "Google Dorking"
description: "Use when searching for an organisation's exposed files, login panels, and sensitive pages using search-engine operators — passive recon that finds what shouldn't be indexed."
description_fr: "À utiliser pour rechercher les fichiers exposés, panneaux de connexion et pages sensibles d'une organisation via des opérateurs de moteur de recherche — une reconnaissance passive qui révèle ce qui n'aurait jamais dû être indexé."
domain: "01-osint-and-reconnaissance"
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

Search engines index far more than an organisation intends — exposed config files, directory listings, login portals, documents with sensitive data, error pages that leak stack traces. Google dorking (search-operator queries) surfaces exactly those. It's pure passive recon: you're querying a search engine, not touching the target, which makes it safe and quiet. This skill covers the operators worth knowing and how to read what they turn up.

### When to use it

Early external recon alongside subdomain enumeration, or as a self-audit ("what has Google indexed about us that shouldn't be public?"). It regularly surfaces the fastest wins — an exposed backup, an admin panel, a document with credentials.

### Procedure

1. **Scope to the target with `site:`** so you only see the organisation's own footprint:
   ```
   site:example.com
   site:example.com -www           # subdomains other than www
   ```
2. **Hunt exposed file types** — backups, configs, spreadsheets, PDFs that often carry sensitive data:
   ```
   site:example.com filetype:sql OR filetype:env OR filetype:bak
   site:example.com filetype:xlsx OR filetype:pdf confidential
   ```
3. **Find login and admin panels** — entry points worth noting for later testing:
   ```
   site:example.com inurl:admin OR inurl:login OR intitle:"dashboard"
   ```
4. **Find directory listings and exposed paths** — open directories leak everything in them:
   ```
   site:example.com intitle:"index of"
   ```
5. **Look for information disclosure** — error pages, config, and keywords that suggest sensitive content:
   ```
   site:example.com intext:"password" OR intext:"api_key"
   site:example.com "sql syntax near" OR "stack trace"
   ```
6. **Broaden beyond the main domain** where in scope — third-party sites (paste sites, code hosts, cloud storage) may index the org's data:
   ```
   "example.com" site:pastebin.com
   ```
7. Verify manually that a hit is real and sensitive before reporting — search caches go stale, and not every match matters.

### Cheatsheet

```
core operators
  site:example.com            limit to the target's domain
  filetype:pdf                specific file types (sql, env, bak, xlsx, log, conf)
  inurl:admin                 term in the URL (login, admin, api, backup)
  intitle:"index of"          directory listings
  intext:"password"           term in page body
  cache:                      google's cached copy (see removed content)
  -www                        exclude a subdomain

high-value dorks
  site:target filetype:env | filetype:sql | filetype:bak      -> configs/backups
  site:target intitle:"index of"                              -> open directories
  site:target inurl:admin | inurl:login                       -> panels
  site:target intext:"api_key" | "BEGIN RSA PRIVATE KEY"      -> secrets
  "target.com" site:pastebin.com | site:github.com            -> off-site leaks

reference: the Google Hacking Database (GHDB) at exploit-db.com/google-hacking-database
```

### Reading the output

- **An exposed config/backup file** (`.env`, `.sql`, `.bak`) = often a direct credential or data leak — high value, verify and report.
- **A directory listing (`index of`)** = everything in that folder is browsable; check what it exposes.
- **A login/admin panel** = an entry point for later testing (default creds, auth weaknesses) — note it, don't attack from a dork.
- **Error pages / stack traces indexed** = information disclosure that aids other attacks (tech stack, paths, sometimes credentials).
- **Off-site hits** (paste sites, public repos) = leaked data outside the org's control — often the most sensitive finds, and a takedown candidate.
- **Stale cache results** = a hit may no longer be live; confirm before acting.

### The fix (for your own exposed content)

- **Remove sensitive files from public paths** and rotate any leaked credentials immediately (a dorked secret is public).
- **Use `robots.txt` and `noindex`** to keep genuinely non-public pages out of indexes — but understand these hide from search engines, not attackers, so don't rely on them for anything sensitive.
- **Disable directory listing** on web servers.
- **Fix information disclosure** — no stack traces or config in production responses (ties into the security-headers and error-handling skills).
- **Request removal** of already-indexed sensitive content via the search engine's removal tools, and address off-site leaks with takedowns.
- **Self-dork regularly** — run these queries against your own domain to catch exposure before someone else does.

### Pitfalls

- **Treating dorking as an attack.** It's passive search — but *acting* on what you find (logging into a panel, downloading someone's data) crosses into unauthorised access. Recon only, unless it's your target and in scope.
- **Trusting stale results.** Caches lag; a hit may be gone or a false positive. Verify.
- **Relying on robots.txt to protect data.** It tells crawlers not to index — it does nothing to stop a human reading the file. Sensitive content must not be public at all.
- **Stopping at the main domain.** The worst leaks are often off-site (paste sites, public repos, cloud storage). Broaden the search where scope allows.

### References

- Google Hacking Database (GHDB) — exploit-db.com/google-hacking-database
- OWASP WSTG-INFO-01 (Search Engine Discovery / Reconnaissance)
- Google Search operators reference

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.