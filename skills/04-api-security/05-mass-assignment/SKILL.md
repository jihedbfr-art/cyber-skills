---
name: mass-assignment
domain: 04-api-security
description: Use when an API binds request data straight onto objects — testing whether you can set fields you shouldn't, like role or ownership, and how to bind safely.
difficulty: intermediate
tags: [owasp-api, mass-assignment, authorization, api]
tools: [burp, curl]
---

## Purpose

Mass assignment happens when an API takes the JSON you send and binds it directly onto a data model, setting whatever properties are present. If the model has sensitive fields — `isAdmin`, `role`, `accountId`, `verified`, `balance` — and the binding doesn't restrict which ones the client may set, you can escalate privilege or tamper with data just by adding a field to the request. This skill covers finding it and binding safely.

## When to use it

APIs that accept object updates or creation (`POST`/`PUT`/`PATCH`), especially those built on frameworks with convenient "bind the whole request to the model" features (Rails, Spring, Django, Laravel, Node ORMs). Common on registration, profile update, and settings endpoints.

## Procedure

1. Learn the **object's full shape.** Read a `GET` response for the object, or the docs, to see every property — including ones the update form doesn't expose (`role`, `isAdmin`, `owner`, `verified`).
2. Capture a legitimate update request and note which fields the client normally sends.
3. **Add a sensitive field** the UI never sends, and see if the server accepts it. The classic is privilege escalation on your own account:
   ```
   curl -X PATCH -H "Authorization: Bearer <token>" https://api.tld/v1/users/me \
     -d '{"name":"me","role":"admin"}'
   ```
   Then re-read the object — did `role` change?
4. Test **ownership/tenant fields**: set `userId`/`accountId`/`ownerId` to another value to reassign or access-cross objects.
5. Test **workflow/state fields**: `verified:true`, `emailConfirmed:true`, `isPaid:true`, `status:"approved"` — anything that skips a process the app controls.
6. Try fields at **creation** too (`POST`), not just update — mass assignment on registration is a frequent way to self-grant a privileged role.
7. Confirm the change actually took effect (re-fetch the object); a field being accepted in the request but ignored is not a finding.

## Cheatsheet

```
1. GET the object -> list every property (esp. ones the form hides)
2. add a sensitive field to a normal update:

  {"name":"x", "role":"admin"}
  {"name":"x", "isAdmin":true}
  {"email":"x", "emailVerified":true}
  {"amount":10, "userId":<other-user>}
  {"plan":"free", "isPaid":true, "status":"approved"}

3. re-GET -> did the privileged field change?  (accepted-but-ignored = not a bug)

also test on POST /register, not just PATCH /me
```

## Reading the output

- **A sensitive field you added actually changing** (role becomes admin, verified becomes true) = confirmed mass assignment, usually privilege escalation or authz bypass. High impact.
- **An ownership field reassigning an object** = you can move records between users/tenants — a serious authorization break.
- **A workflow field letting you skip a step** (self-verify, self-approve, mark paid) = business-logic bypass with direct impact.
- **The field accepted in the request but not reflected on re-fetch** = the server ignored it; not exploitable, don't report it.

## The fix

Control exactly which fields the client is allowed to bind — never bind the whole request onto the model:

- **Allowlist bindable fields** per endpoint (Rails strong parameters, a DTO/input type in Spring/.NET, explicit `pick()`/serializer `fields` in Node/Django). The model's sensitive fields simply aren't in the allowlist, so the client can't set them.
- **Separate input models from persistence models.** Bind the request to an input DTO with only the fields a client may send, then map the safe fields onto the entity server-side.
- **Set sensitive fields server-side only** — role, ownership, verification, and state come from server logic and the authenticated identity, never from the request body.
- Add tests asserting that sending a forbidden field has no effect, so a future refactor to "bind everything" is caught.

## Pitfalls

- **Blocklisting instead of allowlisting.** Blocking `isAdmin` misses `role`, `is_admin`, `admin`, and the next field someone adds. Allowlist what's permitted, deny the rest by default.
- **Binding the ORM model directly.** Convenient framework auto-binding is exactly what creates this — use an input DTO.
- **Fixing update but not create.** Registration endpoints are a favourite for self-granting privilege. Cover `POST` too.
- **Reporting accepted-but-ignored fields.** Confirm the change persisted before calling it a finding.

## References

- OWASP API Security Top 10 — API6:2023 (and API3, property-level authorization)
- CWE-915 (Improperly Controlled Modification of Dynamically-Determined Object Attributes)
- OWASP Mass Assignment Cheat Sheet
