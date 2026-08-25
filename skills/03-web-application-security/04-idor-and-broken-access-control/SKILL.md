---
format: "v2"
name: "idor-and-broken-access-control"
title: "Idor And Broken Access Control"
title_fr: "IDOR et contrôle d'accès défaillant"
description: "Use when testing whether a web app enforces access control server-side — horizontal and vertical privilege checks, function-level auth, and forced browsing — plus the fix."
description_fr: "À utiliser pour vérifier qu'une application web applique bien le contrôle d'accès côté serveur — contrôles de privilèges horizontaux et verticaux, autorisation au niveau des fonctions, et navigation forcée — ainsi que la correction."
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

Broken access control sits at the top of the OWASP Top 10 for a reason: it's everywhere and it's high-impact. The app knows who you are but doesn't consistently check what you're allowed to touch. This skill covers the whole family — object-level (IDOR), function-level, and forced browsing — in a web-app context, and how to enforce authorisation properly.

The API-specific object-level case has its own deeper skill; this one takes the web-app view across all three axes.

### When to use it

On any authenticated app. It's the class of bug you can find without special tools — just two accounts, a proxy, and patience.

### The three axes

- **Horizontal** — reach another user's data at your own privilege level (your `/account/1002` → their `/account/1001`).
- **Vertical** — reach a higher privilege level (a normal user hitting `/admin/*`).
- **Function-level / forced browsing** — the UI hides a capability but the endpoint still answers (the "delete" button is gone for you, the `DELETE` route isn't).

### Procedure

1. Set up two accounts — a low-priv user A and, ideally, an admin B. Log in as A in your proxy and browse normally to collect real requests and IDs.
2. **Horizontal:** take a request that returns your data, change the identifier, replay with A's session. Someone else's data back = IDOR.
   ```
   curl -b "session=A" https://app.tld/api/account/1001
   ```
3. **Vertical:** as A, request admin functionality directly by URL. The UI won't link it; the route may still work:
   ```
   curl -b "session=A" https://app.tld/admin/users
   ```
4. **Function-level:** replay an admin action captured from B's session, but with A's cookie. If it succeeds, the action isn't checking the caller's role:
   ```
   curl -b "session=A" -X POST https://app.tld/api/users/55/promote
   ```
5. Test **parameter- and method-based** bypasses: does adding `?admin=true`, changing `role` in a body, or switching `GET` to `POST` slip past a check that only guards one shape?
6. Check that **logout and privilege changes take effect** — an old session or token that still works after a role downgrade is its own access-control bug.

### Cheatsheet

```bash
curl -b "s=A" https://app.tld/orders/1000
curl -b "s=A" https://app.tld/orders/1001

curl -b "s=A" https://app.tld/admin/
curl -b "s=A" https://app.tld/api/admin/config

curl -b "s=A" -X DELETE https://app.tld/api/users/42
curl -b "s=A" -X POST https://app.tld/api/users/42/role -d '{"role":"admin"}'

ffuf -b "s=A" -u https://app.tld/FUZZ -w paths.txt -mc 200,302
```

### Reading the output

- **`200` + data/action that belongs to someone else or to a higher role** = broken access control. The successful status is the confirmation.
- **`403`/`401` on the cross-boundary attempt** = the check is present for that route — keep testing others, because coverage is usually inconsistent.
- **A `GET` blocked but the `POST` allowed** (or vice versa) points to a check bolted onto one method, not the operation.
- **An admin page returning its shell but no data** may still leak structure; confirm whether the data endpoints behind it are also guarded.

### The fix

Enforce authorisation **server-side, on every request, from the session identity** — never from a value the client can set, never from UI state.

- Default deny. A route with no explicit authorisation decision should reject, not allow.
- Check ownership/role at the point of data access, and prefer queries scoped to the caller so "forgot to check" can't return another tenant's rows.
- Centralise it — a policy layer, middleware, or framework guard — so new endpoints inherit enforcement instead of re-implementing (and re-forgetting) it.
- Don't rely on unguessable IDs, hidden menus, or disabled buttons; those are UX, not access control.
- Invalidate sessions and re-check permissions on privilege changes and logout.

### Pitfalls

- **Testing one axis.** Teams that get horizontal right often miss vertical or function-level. Cover all three.
- **Client-side "security".** A greyed-out button proves nothing about the endpoint behind it.
- **Only checking read.** Write and delete are where the damaging IDORs live and where checks lapse most.
- **Assuming a WAF or gateway handles it.** Authorisation is business logic; it belongs in the app, per object.

### References

- OWASP Top 10 A01:2021 Broken Access Control
- OWASP WSTG-ATHZ (Authorization Testing)
- CWE-284, CWE-639, CWE-285

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.