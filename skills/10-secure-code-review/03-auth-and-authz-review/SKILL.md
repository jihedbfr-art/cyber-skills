---
format: "v2"
name: "auth-and-authz-review"
title: "Auth And Authz Review"
title_fr: "Revue de l'authentification et des autorisations"
description: "Use when reviewing access-control logic — reading authn/authz code for the gaps that let a request act as someone it isn't, or reach something it shouldn't."
description_fr: "À utiliser lors de la revue de la logique de contrôle d'accès — en lisant le code d'authentification et d'autorisation pour détecter les failles qui permettent à une requête d'usurper une identité ou d'atteindre une ressource qui ne lui est pas destinée."
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

Injection bugs are loud; access-control bugs are quiet. Nothing crashes, no payload looks weird — the code just checks the wrong thing, or nothing, and a user reads another user's data. This skill is about reading auth code specifically, because taint tracking won't find these: the input is perfectly valid, it's the *decision* that's wrong.

Authentication answers "who are you"; authorization answers "are you allowed to do this". Most real findings are in the second.

### When to use it

Reviewing any endpoint, controller, service method, or middleware that gates access — which is nearly all of them. Do this pass separately from the injection pass; you're looking for absence of a check, which is harder to see than a bad line.

### What to actually look for

**The missing check.** The single most common flaw: an endpoint that authenticates the user but never verifies the object belongs to them. `GET /orders/{id}` that loads the order by id and returns it — without `order.ownerId == currentUser.id`. That's IDOR, and it's invisible unless you ask "who owns this row?" for every fetch by id. Read each data access and ask: could I put someone else's id here?

**Trusting client-supplied identity.** A `userId` in the request body, a role in a cookie, a `X-User-Id` header the gateway was *supposed* to strip. If the authorization decision reads identity from something the client can set, it's broken. Identity comes from the authenticated session/token, never from the request payload.

**Check-then-use gaps.** The authz check runs against one value, the operation uses another. Or the check is in the controller but a second code path reaches the service directly and skips it.

**Broken function-level access.** Admin routes protected only by "the link isn't in the UI." Grep the admin/internal handlers and confirm each has a server-side role check, not just a hidden button.

**Verb and path confusion.** A framework rule guarding `POST /admin` but not `PUT`, or a case/trailing-slash mismatch that dodges the filter.

In a Spring codebase, `@PreAuthorize` and method security are where I focus, but the trap is a service method with no annotation that's reachable from more than one controller — the guarded controller lulls you while the unguarded one walks straight in. Check the service layer, not just the entry points.

### Procedure

1. List the endpoints/handlers and, for each, name the resource it touches and whose it is.
2. For every fetch/update/delete *by id*, find the ownership or tenant check. No check on a by-id operation = probable IDOR; write the path.
3. Trace where identity comes from in each authz decision. If it's the request body/params/headers rather than the session/token, flag it.
4. Find the privileged operations (admin, billing, user management, exports) and confirm a server-side role/permission gate on each.
5. Check for the second door: can the guarded logic be reached through another controller, a batch job, a GraphQL resolver, an internal API, without the guard?

### Cheatsheet

```bash
rg -n 'findById|getById|\.get\(id|WHERE id ?=|/:id|\{id\}'
rg -n 'req\.body\.(user|role)|getParameter\("(userId|role)"|headers\[.?x-user'
rg -n '@PreAuthorize|@Secured|hasRole|hasPermission|isAuthenticated|requireRole|can\?'
```

### Reading it

- **By-id operation, no ownership predicate** → IDOR/BOLA. The most valuable finding class in this domain.
- **Role read from token/session, checked server-side** → the correct shape.
- **Role or userId read from request, or checked only client-side** → broken; trivially forged.
- **One entry point guarded, another reaching the same logic unguarded** → the guard is decorative. Report the unguarded path.

### The fix

Enforce authorization at the layer every path goes through — ideally the service/domain layer, not per-controller, so a new endpoint can't forget it. Derive identity from the authenticated principal only. For object access, make ownership/tenant scoping part of the query itself (`WHERE id = ? AND tenant_id = ?`) so a wrong id returns nothing instead of relying on a separate check that can be skipped. Deny by default.

### Pitfalls

- **Reviewing only controllers.** The gap is often a shared service reachable from several places.
- **Confusing authn for authz.** "The user is logged in" is not "the user may touch *this* record."
- **Assuming the ORM/tenant filter is global.** Confirm it, don't hope it. Global filters get bypassed by native queries.
- **Skipping the boring CRUD.** IDOR lives in exactly the boring by-id endpoints people don't reread.

### References

- OWASP A01:2021 Broken Access Control
- CWE-639 (IDOR), CWE-285 (Improper Authorization), CWE-306 (Missing Authentication)
- See also domain 03 `idor-and-broken-access-control` and domain 04 `broken-object-level-authorization`

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.