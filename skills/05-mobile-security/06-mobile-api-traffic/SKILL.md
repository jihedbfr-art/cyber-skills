---
name: mobile-api-traffic
domain: 05-mobile-security
description: Use when testing the backend a mobile app talks to — intercepting and testing the API, because the real attack surface is often the server, not the app on the device.
difficulty: intermediate
tags: [mobile, api, traffic, backend, interception]
tools: [burp, mitmproxy]
---

## Purpose

A mobile app is a client, and clients can be manipulated — but the data and logic live on the backend API. Much of a mobile app's real attack surface is that API, and the most impactful mobile findings often come from testing the server the app talks to, not the app itself. This skill covers intercepting and testing mobile API traffic — treating the mobile backend as the API it is, with the full API-security toolkit applied.

## When to use it

After you can see the app's traffic (past any pinning — the pinning-bypass skill). It's frequently the highest-value part of a mobile assessment, because backend flaws (broken authorization, injection, excessive data exposure) have server-side impact affecting all users, not just the one device.

## Procedure

1. **Get visibility into the traffic.** Route the app through an intercepting proxy (Burp, mitmproxy) with its CA installed, bypassing certificate pinning if present (the pinning skill). Now you can see and modify the requests the app makes to its backend.
2. **Map the API.** Exercise the app fully and catalogue the endpoints it calls, the parameters, the authentication mechanism, and the data exchanged. The app's traffic reveals the API's shape — often an undocumented API you now understand.
3. **Apply the full API-security toolkit — this is the point.** The mobile backend is an API, so all the API-security skills apply: test for **broken object-level authorization (BOLA/IDOR)** — the top API flaw, and rife in mobile backends — by manipulating object IDs to reach other users' data; **broken authentication**, **excessive data exposure** (mobile APIs often over-return, trusting the app to filter — but you see the raw response), **mass assignment**, **injection**, and **rate limiting**. The API-security domain covers each.
4. **Exploit the client-manipulation angle.** The app enforces things client-side (input validation, hidden fields, business rules) that the server may trust — bypass the client by manipulating requests directly. A value the app never lets the user change, changed in the request, tests whether the server validates it. Client-side controls the server relies on are a common mobile-backend flaw.
5. **Check what the API over-returns.** Mobile APIs frequently send more than the app displays (extra fields, other users' data, internal flags) trusting the client to show only some — but the raw response is visible in the proxy (the excessive-data-exposure skill). Read the full responses.
6. **Test authentication and session handling** as sent by the app — token validation, expiry, and whether the API properly authenticates every request (the API broken-authentication skill).
7. **Report backend findings by server-side impact** — a BOLA or injection in the mobile API affects all users through the server, not just the tested device; these are typically the highest-severity mobile findings.

## Cheatsheet

```
the app = a CLIENT (manipulable) ; data + logic = the BACKEND API
  much of the real attack surface (+ highest-impact findings) = the SERVER, not the app

1. VISIBILITY: route through proxy (Burp/mitmproxy) + CA + bypass pinning -> see/modify requests
2. MAP the API: endpoints, params, auth, data (traffic reveals the shape — often undocumented API)
3. APPLY THE FULL API-SECURITY TOOLKIT (the point — it IS an API):
     BOLA/IDOR (top flaw, rife in mobile backends — manipulate object IDs -> other users' data)
     broken auth | EXCESSIVE DATA EXPOSURE (APIs over-return, you see RAW response)
     mass assignment | injection | rate limiting
4. CLIENT-MANIPULATION: app enforces client-side (validation/hidden fields/rules) server may TRUST
     -> bypass client, manipulate request directly (server validates it?)
5. read FULL responses (over-returned fields/data the app hides but proxy shows)
6. auth/session as sent (token validation, expiry, per-request auth)
report by SERVER-SIDE impact (BOLA/injection in mobile API = ALL users, not one device)
```

## Reading the traffic

- **BOLA/IDOR in the mobile backend** (changing an object ID reaches another user's data) = the top API flaw and extremely common in mobile backends; a server-side impact affecting all users. Usually the highest-value mobile finding — test it thoroughly.
- **The API over-returning data** (fields, other users' info, internal flags the app hides) = excessive data exposure; the app filters for display, but the raw response is visible in the proxy. Read full responses, don't trust the app's view.
- **A client-side-enforced rule the server trusts** (validation, a hidden field, a business rule) bypassed by manipulating the request = the server relies on the client, which the attacker controls. A common, impactful mobile-backend flaw.
- **The API accepting a request with a tampered value the app never sends** = the server isn't validating server-side; client-side enforcement isn't security. A finding.
- **Weak per-request authentication** = the API version of broken authentication; test tokens as the app sends them.
- **Backend flaws (BOLA, injection, exposure)** = server-side impact across all users; these outrank device-local app findings in severity. Prioritise the backend.

## Pitfalls

- **Focusing on the app, not the backend.** The app is a manipulable client; the highest-impact findings are usually server-side (BOLA, injection, exposure affecting all users). Treat the mobile backend as the API it is and test it fully.
- **Trusting the app's displayed data.** APIs over-return; the app filters for display but the raw response (visible in the proxy) may contain other users' data or sensitive fields. Read full responses.
- **Assuming client-side controls are security.** Validation, hidden fields, and rules the app enforces are bypassable by manipulating requests; the server must validate independently. Test whether it does.
- **Not applying the API-security toolkit.** The mobile backend is an API — BOLA, broken auth, mass assignment, injection, and rate limiting all apply. Skipping them misses the real attack surface.
- **Under-rating backend findings.** A device-local app issue affects one device; a backend BOLA affects all users. Prioritise by server-side impact.

## References

- The API-security domain (BOLA, broken-authentication, excessive-data-exposure, mass-assignment, etc.) — apply it all
- The ssl-pinning-bypass skill (to get traffic visibility) and OWASP MASTG (network testing)
- Burp Suite and mitmproxy documentation
- OWASP API Security Top 10
