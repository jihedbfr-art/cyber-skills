---
name: broken-object-level-authorization
domain: 04-api-security
description: Use when testing whether an API lets one user access another user's objects by changing an ID — the BOLA/IDOR flaw that tops the OWASP API list, plus the server-side fix.
difficulty: intermediate
tags: [owasp-api, bola, idor, authorization, access-control]
tools: [burp, curl, ffuf]
---

## Purpose

Broken Object Level Authorization is the API's most common serious bug: the endpoint authenticates *who you are* but never checks whether *this object is yours*. Swap the ID in `/api/orders/1042` for `1041` and you read someone else's order. This skill covers finding it, proving it cleanly, and fixing it where it belongs — server-side, per request.

## When to use it

Every API endpoint that takes an object identifier — path (`/users/{id}`), query (`?account=`), body field, or header. Especially anything returning or modifying data tied to a specific user, order, document, or tenant. If the client sends an ID and gets data back, test it.

## Procedure

1. Authenticate as a normal, low-privilege user and browse the app so the API issues requests with real object IDs. Capture them in your proxy.
2. Pick a request that fetches one of *your* objects, e.g. `GET /api/v1/invoices/5001`. Note the ID and that it returned your data.
3. Change only the ID and replay with your same valid token. If you get back an object that isn't yours, that's BOLA:
   ```
   curl -H "Authorization: Bearer <your-token>" https://api.tld/v1/invoices/5000
   ```
4. Confirm it's an authorization gap and not just enumerable-but-empty. The tell is a `200` with someone else's real data, versus a proper `403`/`404`.
5. Test every method, not just GET. Write access is worse — can you `PUT`/`DELETE` an object you don't own?
   ```
   curl -X DELETE -H "Authorization: Bearer <your-token>" https://api.tld/v1/invoices/5000
   ```
6. For a cleaner proof and to gauge blast radius, use two accounts: create an object as user A, then access it as user B's token. Success across the account boundary is unambiguous.
7. If IDs are sequential, a small controlled sweep shows scale — but pull only enough to prove the finding, not the whole table:
   ```
   ffuf -H "Authorization: Bearer <your-token>" -u https://api.tld/v1/invoices/FUZZ -w ids.txt -mc 200
   ```

## Cheatsheet

```bash
# baseline: your own object
curl -H "Authorization: Bearer $A" https://api.tld/v1/users/1001

# horizontal: someone else's object, your token
curl -H "Authorization: Bearer $A" https://api.tld/v1/users/1002

# two-account proof (create as A, read as B)
curl -H "Authorization: Bearer $A" -X POST https://api.tld/v1/docs -d '{"title":"t"}'
curl -H "Authorization: Bearer $B" https://api.tld/v1/docs/<id-from-A>

# other places IDs hide
?account_id=1002        # query string
X-Account-Id: 1002      # header the server trusts
{"userId": 1002}        # request body
```

Watch for GUIDs instead of integers — harder to guess but *not* a fix. If the object leaks through another endpoint or a predictable pattern, BOLA still applies.

## Reading the output

- **`200` + data that isn't yours** = confirmed BOLA. The status is the giveaway: the server processed the request and handed over the object.
- **`403` / `401`** = the check exists. Good — but test other methods and other endpoints; authorization is often enforced on read and forgotten on update/delete.
- **`404` for another user's real object** = the safe pattern (don't confirm existence to unauthorised callers). Verify the object truly exists via the owner's account before calling it fixed.
- **A GUID that only works via one path** isn't safety — check whether the ID leaks elsewhere (list endpoints, referrers, other users' data).

## The fix

Add an ownership check on **every** object access, enforced on the server using the authenticated identity from the token — never trusting an ID or owner field the client sent:

```java
// vulnerable — fetches by id, no owner check
Invoice inv = invoiceRepo.findById(id);
return inv;

// fixed — the query is scoped to the caller
Invoice inv = invoiceRepo.findByIdAndOwner(id, currentUser.getId());
if (inv == null) return notFound();   // 404, don't confirm existence
return inv;
```

Principles that make it stick:

- Derive the user from the session/token server-side; ignore any `userId`/`ownerId` in the request as an authorization input.
- Scope the data-access query itself to the owner (or tenant) rather than fetching first and checking after — the check can't be forgotten if the query can't return others' rows.
- Enforce it in a central layer (policy/guard/middleware), so a new endpoint doesn't reintroduce the gap.
- Random GUIDs and hashed IDs raise the guessing cost but are not access control. Keep the check regardless.

## Pitfalls

- **Testing GET only.** The nastiest BOLAs are on `PUT`/`PATCH`/`DELETE`, where the check is skipped more often.
- **Dumping data to "prove impact."** Pull the minimum to demonstrate it. Mass-extracting real user records can breach the very scope you're testing under.
- **Assuming GUIDs are safe.** Unguessable is not unauthorised. The bug is the missing check, not the ID format.
- **Only checking horizontal access.** Also test vertical — can a user reach an admin-only object by ID?

## References

- OWASP API Security Top 10 — API1:2023 Broken Object Level Authorization
- OWASP WSTG-ATHZ-04 (IDOR)
- CWE-639, CWE-284
