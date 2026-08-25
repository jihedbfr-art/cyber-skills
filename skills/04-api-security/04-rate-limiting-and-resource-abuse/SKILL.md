---
format: "v2"
name: "rate-limiting-and-resource-abuse"
title: "Rate Limiting And Resource Abuse"
title_fr: "Limitation de débit et abus de ressources"
description: "Use when testing whether an API limits how often and how heavily it can be called — the missing controls that enable brute force, scraping, and cost/DoS attacks."
description_fr: "À utiliser pour tester si une API limite la fréquence et le volume de ses appels — les contrôles manquants qui rendent possibles le brute force, le scraping et les attaques par coût ou déni de service."
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

Without limits, an API lets a single client do unlimited work: brute-force credentials, scrape the whole dataset, or drive up compute and bandwidth until the service (or the bill) falls over. This skill covers testing for missing or weak rate limiting and resource controls, and how to add them.

### When to use it

Any exposed API, especially endpoints that are expensive (search, export, report generation), sensitive (login, OTP, password reset), or that return lots of data. Also any endpoint whose cost scales with a client-supplied parameter (page size, date range).

Do this within scope and gently — the whole point is resource exhaustion, so uncontrolled testing can cause the very outage you're checking for. Use small, controlled bursts.

### Procedure

1. **Check for a basic rate limit.** Send a controlled burst of requests to an endpoint and watch for `429 Too Many Requests` or throttling. No slowdown after many requests = no limit:
   ```
   for i in $(seq 1 100); do curl -s -o /dev/null -w "%{http_code}\n" https://api.tld/v1/search?q=x; done | sort | uniq -c
   ```
2. **Test the sensitive endpoints specifically** — login, OTP verification, password reset, coupon redemption. These need strict limits because unlimited attempts = brute force. Confirm lockout/throttling exists.
3. **Test limit bypasses** — is the limit per-IP only (rotate IP / add `X-Forwarded-For`), per-token (use several), or missing on some routes? A limit on `/login` but not `/api/v2/login` is a common gap.
4. **Resource-cost abuse (unrestricted consumption).** Can the client ask for arbitrarily expensive work? Huge `page_size`/`limit`, unbounded date ranges, deeply nested queries, or large uploads — each turns one request into heavy server work:
   ```
   curl "https://api.tld/v1/report?limit=1000000&from=1970-01-01"
   ```
5. **Cost attacks** on pay-per-use backends — endpoints that trigger SMS, email, or third-party API calls (and thus real money) with no cap. Flag these as financial-impact even at low request volume.
6. Record the impact class: brute force, scraping/data harvesting, DoS, or cost — the fix differs slightly per class.

### Cheatsheet

```bash
seq 1 100 | xargs -I{} -P10 curl -s -o /dev/null -w "%{http_code}\n" api.tld/v1/x | sort | uniq -c

-H "X-Forwarded-For: 1.2.3.4"    # spoofed client IP
rotate source IPs / multiple tokens
try /api/v2/... variant that may lack the limit

?limit=100000   ?page_size=99999   ?from=1900-01-01   deeply nested GraphQL
endpoints that send SMS/email/3rd-party calls  -> cost attack
```

### Reading the output

- **No `429` after a sustained burst** = no rate limiting; brute force, scraping, and DoS are all on the table. Rate by which the endpoint enables.
- **Limit on the main route but not a variant/version** = the control is bypassable and effectively absent.
- **A per-IP-only limit** falls to IP rotation or `X-Forwarded-For` spoofing — note it as weak, not present.
- **An endpoint that returns unbounded data or does unbounded work from one request** = resource exhaustion regardless of request count — one call can hurt.
- **An unmetered endpoint that costs money per call** (SMS/email/paid API) = financial DoS; treat as high impact even at low volume.

### The fix

- **Rate limit every endpoint**, keyed on both IP and authenticated identity (token/user), and return `429` with a `Retry-After`. Enforce it at the gateway so no route is missed.
- **Stricter limits on sensitive endpoints** (login, OTP, reset) plus lockout/backoff — this is also the brute-force defence.
- **Cap resource-cost parameters**: maximum `page_size`, bounded date ranges, pagination required, query depth/complexity limits (especially GraphQL), and upload size limits.
- **Quotas and spend caps** on endpoints that trigger paid downstream actions, so a loop can't run up the bill.
- **Timeouts and payload limits** so a single slow/large request can't tie up resources.
- Monitor and alert on abnormal request rates and cost spikes — detection backstops the limits.

### Pitfalls

- **Per-IP limiting only.** Trivially bypassed by rotation; combine with per-identity limits.
- **Limiting the obvious endpoint, not the variants.** `/login` guarded, `/api/v2/login` open. Cover all routes and versions.
- **Ignoring resource cost.** A rate limit on request *count* doesn't help if one request can request a million rows. Cap the parameters too.
- **Testing too aggressively.** You can cause the outage you're probing for. Small controlled bursts, within scope.

### References

- OWASP API Security Top 10 — API4:2023 Unrestricted Resource Consumption
- CWE-770 (Allocation of Resources Without Limits), CWE-307 (brute force)
- OWASP — Denial of Service and rate limiting guidance

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.