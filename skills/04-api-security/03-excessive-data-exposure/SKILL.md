---
format: "v2"
name: "excessive-data-exposure"
title: "Excessive Data Exposure"
title_fr: "Exposition excessive de données"
description: "Use when an API returns more data than the client needs — testing whether responses leak fields the user shouldn't see, and how to filter at the server."
description_fr: "À utiliser quand une API renvoie plus de données que nécessaire au client — pour tester si les réponses exposent des champs que l'utilisateur ne devrait pas voir, et pour les filtrer côté serveur."
domain: "04-api-security"
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

A common API anti-pattern: the endpoint returns the whole object and trusts the client (the app UI) to display only the appropriate fields. But the API response is right there in the network tab — anyone can read the fields the UI hides. This skill covers finding those over-returning endpoints and fixing them by filtering server-side.

### When to use it

Any API that returns objects — user profiles, orders, records. Especially where a mobile/web client shows a subset of what the endpoint actually sends. It's easy to find (just read responses) and often leaks PII or internal fields.

### Procedure

1. Call the endpoints and **read the full raw response**, not what the app displays. Compare the fields returned against what the UI actually uses:
   ```
   curl -H "Authorization: Bearer <token>" https://api.tld/v1/users/me | jq
   ```
2. Look for fields the client shouldn't get: password hashes, internal flags, other users' PII, `isAdmin`, tokens, full records when only a name was needed. If the app shows a name but the response includes an email and phone, that's exposure.
3. Check **list/collection endpoints** — they often over-return per item, multiplying the leak across every record.
4. Test **object endpoints for other users** in combination with authorisation checks (ties into BOLA) — over-exposure plus weak authz means bulk PII harvesting.
5. Look for **debug/verbose fields** left in responses (stack traces, internal IDs, SQL) — information leakage that helps other attacks.
6. Confirm the extra data is genuinely sensitive/unintended before reporting — some fields are meant to be there.

### Cheatsheet

```bash
curl -s -H "Authorization: Bearer $T" https://api.tld/v1/users/me | jq 'keys'

password / passwordHash / salt
ssn / dob / fullAddress / phone   (when the view only needs a name)
isAdmin / role / internalNotes / accountBalance (of others)
apiKey / token / secret
stackTrace / sqlQuery / internalId

curl -s .../v1/users | jq '.[0] | keys'
```

### Reading the output

- **Sensitive fields present in the response but hidden by the UI** = excessive data exposure. The client-side hiding is not a control; the data already left the server.
- **PII across a list endpoint** = the highest-impact version — one request harvests many records' worth of sensitive data.
- **Internal/debug fields** (stack traces, SQL, internal IDs) = information leakage that aids further attacks even if not directly sensitive.
- **Fields that are supposed to be there** (the user's own email on their own profile settings page) = not a finding; judge by whether the caller should legitimately receive it.

### The fix

Filter on the server, by design, so the API can only ever return what the caller is entitled to:

- **Explicit response schemas / DTOs** — define exactly which fields each endpoint returns, rather than serialising the whole database object. Never `return user` straight from the ORM.
- **Never rely on the client to hide data.** The response is visible; if the client shouldn't see it, don't send it.
- **Field-level authorisation** where different callers get different fields (a user sees their own email, an admin view is separate and access-controlled).
- **Strip debug/verbose output** in production responses (no stack traces, no internal identifiers).
- Review responses as part of API design, and add tests that assert an endpoint's response shape so a future change can't quietly widen it.

### Pitfalls

- **Trusting the UI to filter.** The single mistake behind this whole class — the raw response is one network-tab click away.
- **Fixing the object endpoint, missing the list.** Collection endpoints leak the same fields times N. Check both.
- **Serialising the ORM model directly.** It returns every column, including ones added later. Use an explicit output shape.
- **Over-reporting.** Not every extra field is sensitive; judge by whether the caller is entitled to it before flagging.

### References

- OWASP API Security Top 10 — API3:2023 (Broken Object Property Level Authorization, which covers excessive exposure)
- CWE-213 (Exposure of Sensitive Information Due to Incompatible Policies)
- OWASP WSTG — information disclosure testing

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.