---
name: api-gateway-hardening
domain: 04-api-security
description: Use when deciding where and how to enforce API security controls — using the gateway as the consistent choke point for auth, rate limits, and schema instead of per-service reinvention.
difficulty: intermediate
tags: [api, gateway, architecture, rate-limiting, authentication]
tools: [kong, nginx, envoy]
---

## Purpose

An API gateway sits in front of your services and every request passes through it — which makes it the natural place to enforce the controls that would otherwise be re-implemented (and re-forgotten) in each service. This skill covers what belongs at the gateway, what doesn't, and how to harden the gateway itself so it doesn't become the single point everyone can attack.

## When to use it

Designing or reviewing an API platform with more than one service behind it. It's the architectural counterpart to the individual API skills: those find bugs in a service, this decides which defences should live centrally.

## What belongs at the gateway (and what doesn't)

**Enforce centrally at the gateway:**
- **Authentication** — validate the token/API key once, reject the unauthenticated at the edge.
- **Rate limiting and quotas** — per-client and per-identity, applied uniformly.
- **TLS termination** and enforcing HTTPS-only.
- **Request schema/size validation** — reject malformed or oversized requests before they reach a service.
- **Basic WAF-style filtering** and IP allow/deny where relevant.
- **Logging and correlation IDs** so every request is traceable.

**Keep in the service (the gateway can't do it right):**
- **Authorization on objects** — BOLA/ownership checks need the business context only the service has. The gateway can check "authenticated"; it can't check "owns this record".
- **Business-logic validation.**

## Procedure

1. **Put authentication at the edge.** The gateway validates the token and passes a verified identity to the service (e.g. a signed header), so services trust the gateway's assertion rather than re-validating — but ensure services only accept that identity *from* the gateway.
2. **Apply rate limiting and quotas** at the gateway, keyed on identity and IP, with sane defaults for every route so a new endpoint is covered automatically.
3. **Validate requests** — enforce a schema, cap body size, restrict methods and content types — rejecting junk before it costs a backend anything.
4. **Terminate TLS and force HTTPS**, with modern cipher config (ties into the TLS skill).
5. **Harden the gateway itself.** It's now a high-value target: lock down its admin API (never expose the Kong/Envoy admin interface publicly — a classic breach), keep it patched, least-privilege its access to backends, and monitor it.
6. **Don't let the gateway become a bypass.** Ensure services aren't reachable directly, skipping the gateway's controls — network policy must force traffic through it.
7. **Keep object-level authorization in the services** — resist the temptation to centralise what needs business context.

## Cheatsheet

```
GATEWAY (central choke point)          SERVICE (needs business context)
  authenticate the caller                authorize the object (BOLA/ownership)
  rate limit + quotas                    business-logic validation
  TLS termination / HTTPS-only           per-record access decisions
  request schema + size + method
  logging + correlation IDs
  IP allow/deny, basic filtering

harden the gateway
  [ ] admin API NOT publicly exposed (the classic mistake)
  [ ] gateway patched + least-privilege to backends
  [ ] services unreachable except via the gateway (no bypass path)
  [ ] gateway access logged + monitored
```

## Reading an architecture for this

- **Auth re-implemented in every service** = drift and gaps; centralise validation at the gateway and let services trust a verified identity from it.
- **A publicly reachable gateway admin API** = critical — it often allows reconfiguring routes and auth. Lock it to an internal network.
- **Services reachable directly, bypassing the gateway** = every central control is optional. Network policy must close the bypass.
- **Object authorization pushed to the gateway** = usually broken, because the gateway lacks the record-level context; that check belongs in the service.
- **No per-route default rate limit** = new endpoints ship unprotected. Defaults at the gateway fix this.

## Pitfalls

- **Exposing the gateway's admin interface.** A recurring real-world breach — the management API on a public port hands over the whole platform.
- **Centralising object-level authz.** The gateway can't make ownership decisions it has no context for; keep BOLA checks in the service.
- **A bypass path to services.** If backends are directly reachable, the gateway's auth and rate limits mean nothing. Enforce gateway-only ingress.
- **Trusting a gateway-set identity header without restricting its source.** If a service accepts that header from anywhere, an attacker spoofs it. Only accept it from the gateway.

## References

- OWASP API Security Top 10 (architecture and defence-in-depth guidance)
- Kong / Envoy / NGINX hardening documentation
- OWASP API Security Cheat Sheet
