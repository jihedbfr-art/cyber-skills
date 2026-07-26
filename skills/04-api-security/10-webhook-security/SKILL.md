---
name: webhook-security
domain: 04-api-security
description: Use when building or reviewing webhook endpoints — verifying that inbound events are authentic and can't be forged, replayed, or used to attack your internal network.
difficulty: intermediate
tags: [api, webhooks, signatures, ssrf, replay]
tools: [curl, burp]
---

## Purpose

A webhook is an endpoint you expose so another service can POST events to you — a payment succeeded, a build finished. The catch: anyone on the internet can also POST to that URL. If you don't verify the sender, an attacker forges events (fake "payment received"), and if you fetch or act on webhook data carelessly, you open SSRF and injection. This skill covers securing both directions of a webhook.

## When to use it

Any endpoint that receives events from a third party (Stripe, GitHub, a payment processor) or that you expose for partners. Also when configuring *outbound* webhooks your app sends, which have their own SSRF angle.

## Procedure (receiving webhooks)

1. **Verify the signature.** Reputable providers sign each request (HMAC over the raw body with a shared secret, in a header like `X-Signature`). Recompute it and compare — reject anything that doesn't match. This is the core control; without it the endpoint trusts anyone:
   ```
   computed = HMAC-SHA256(shared_secret, raw_request_body)
   reject if computed != header_signature
   ```
2. **Verify over the raw, unparsed body.** Signature verification must use the exact bytes received; parsing then re-serialising JSON changes the bytes and breaks (or bypasses) the check.
3. **Prevent replay.** A captured valid webhook can be re-sent. Check the timestamp in the signed payload is recent, and track event IDs to reject duplicates — so a replayed "payment succeeded" doesn't credit twice.
4. **Use constant-time comparison** for the signature to avoid timing side channels.
5. **Treat the payload as untrusted input.** Even authentic webhooks carry data that flows into your app — apply the usual injection defences, and never pass a URL or ID from the payload into a server-side fetch without validation (SSRF).
6. **Scope what a webhook can do.** A webhook event should trigger a bounded action, not arbitrary operations; validate that the event type and data match what you expect.

## Procedure (sending outbound webhooks — the SSRF angle)

7. If your app lets users configure a URL you'll POST to, that's server-side request forgery by design — validate the destination against an allowlist / block internal ranges, exactly as in the SSRF skill, or a user points your webhook at `169.254.169.254` and your internal services.

## Cheatsheet

```
receiving — verify authenticity
  [ ] recompute HMAC over the RAW body, compare (constant-time)
  [ ] reject on mismatch — no signature, no trust
  [ ] check signed timestamp is recent (anti-replay)
  [ ] track event id -> reject duplicates (idempotency)
  [ ] treat payload as untrusted (injection, SSRF on any URL/id)

sending — SSRF angle
  [ ] user-supplied target URL? allowlist / block internal ranges / re-check resolved IP

quick test (as attacker)
  POST a forged event with no/invalid signature -> is it accepted? (should 401)
  replay a captured valid event -> is it processed twice?
```

## Reading the output

- **A forged event (no or wrong signature) accepted and acted on** = the endpoint trusts anyone — critical, especially for payment/entitlement webhooks where a fake event has direct financial impact.
- **A replayed valid event processed twice** = missing replay protection; "payment succeeded" credited repeatedly. Idempotency and timestamp checks close it.
- **Signature verified over parsed-then-reserialised JSON** = subtly broken; the check may pass on manipulated payloads. Verify the raw bytes.
- **A user-configurable outbound webhook reaching internal services/metadata** = SSRF; treat via the SSRF/metadata skills.
- **Non-constant-time comparison** = a timing oracle on the signature; low-likelihood but a real weakness in a security check.

## The fix

- **Always verify the signature** over the raw body with the provider's shared secret, constant-time, and reject on failure. This is non-negotiable for any state-changing webhook.
- **Add replay protection**: reject stale timestamps and deduplicate on event ID (idempotency keys), so re-sent events are harmless.
- **Validate and sanitise the payload** before acting; don't trust field values just because the signature is valid, and never fetch a URL from the payload without SSRF checks.
- **Allowlist outbound targets** for user-configured webhooks and block internal ranges.
- **Scope the action** a webhook can trigger, and log webhook events for audit.

## Pitfalls

- **No signature verification.** The default failure — an open POST endpoint that trusts its input. Fake events follow.
- **Verifying the parsed body, not the raw bytes.** Re-serialisation changes the bytes; the check becomes unreliable or bypassable.
- **No replay/idempotency protection.** A valid event captured once can be replayed for repeated effect — costly for financial webhooks.
- **Trusting authentic payloads blindly.** Signed doesn't mean safe to fetch or execute; SSRF and injection still apply.
- **Ignoring the outbound direction.** User-set webhook URLs are SSRF unless the destination is validated.

## References

- OWASP API Security Top 10 and SSRF Prevention Cheat Sheet
- Provider webhook security docs (Stripe, GitHub) as reference implementations
- CWE-345 (Insufficient Verification of Authenticity), CWE-294 (replay), CWE-918 (SSRF)
