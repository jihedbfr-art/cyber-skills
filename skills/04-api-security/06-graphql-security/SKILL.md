---
name: graphql-security
domain: 04-api-security
description: Use when testing a GraphQL API — introspection exposure, query depth/complexity abuse, batching attacks, and authorization gaps unique to the graph model — plus the fixes.
difficulty: intermediate
tags: [owasp-api, graphql, api, dos, authorization]
tools: [burp, graphql-cop, altair]
---

## Purpose

GraphQL gives the client a single endpoint and a lot of power: ask for exactly the fields you want, follow relationships, and batch queries. That flexibility is also the attack surface — introspection maps the whole schema, nested queries exhaust the server, and authorization has to be enforced per field, not per endpoint. This skill covers testing GraphQL's specific weaknesses and hardening it.

## When to use it

Any GraphQL endpoint (usually `/graphql`). The web and API skills still apply to the underlying resolvers (injection, BOLA, etc.); this skill adds the GraphQL-specific issues on top.

## Procedure

1. **Introspection.** GraphQL can describe its own schema. If introspection is enabled in production, you get the full map of types, fields, and mutations — an attacker's blueprint:
   ```
   curl -X POST https://api.tld/graphql -H 'Content-Type: application/json' \
     -d '{"query":"{__schema{types{name fields{name}}}}"}'
   ```
   A full schema back = introspection exposed.
2. **Depth / complexity abuse (DoS).** Relationships can be nested to force enormous work from one query. Test whether deep nesting is allowed (gently — this is a resource attack):
   ```
   { user { friends { friends { friends { friends { id } } } } } }
   ```
3. **Batching attacks.** GraphQL can accept an array of operations or aliased duplicates in one request — a way to brute-force or bypass per-request rate limits by doing many operations in one call:
   ```
   { a: login(pw:"1"){ok} b: login(pw:"2"){ok} c: login(pw:"3"){ok} }
   ```
4. **Field-level authorization.** The graph lets a caller reach related objects. Test whether you can traverse to data you shouldn't via a relationship (e.g. `me { company { allEmployees { salary } } }`) — authz must be checked on each field/resolver, and often isn't.
5. **Injection through arguments** — resolvers hit databases; the usual injection tests apply to GraphQL arguments (hand off to the SQLi/NoSQLi skills).
6. Use an automated auditor to sweep the common issues, then verify by hand:
   ```
   graphql-cop -t https://api.tld/graphql
   ```

## Cheatsheet

```graphql
# introspection (should be OFF in prod)
{ __schema { types { name } } }
{ __type(name:"User"){ fields { name } } }

# depth abuse (DoS) — nest relationships deeply
{ a { b { c { d { e { id }}}}} }

# batching / aliasing (bypass per-request limits, brute force)
{ q1: login(pw:"a"){ok}  q2: login(pw:"b"){ok}  q3: login(pw:"c"){ok} }

# authz traversal — reach data via a relationship
{ me { organization { members { email ssn } } } }
```
```bash
graphql-cop -t https://api.tld/graphql   # automated common-issue sweep
```

## Reading the output

- **Full schema returned by introspection** = information exposure; not a vuln alone but it hands attackers the map. Usually a finding to disable in prod.
- **A deeply nested query executing without limit** = complexity/depth DoS; one request can exhaust the backend.
- **Aliased/batched operations all processed** = per-request rate limits are bypassable; combine with the login case for scalable brute force.
- **Reaching another user's/tenant's data through a relationship** = broken field-level authorization — the GraphQL flavour of BOLA, and often the highest-impact finding.
- **Injection in a resolver argument** = same severity as the underlying injection class; treat via that skill.

## The fix

- **Disable introspection in production** (leave it on in dev only), and disable verbose field suggestions that leak schema hints.
- **Enforce query depth and complexity limits** — cap nesting depth and assign cost to fields, rejecting queries over a budget. This is the core DoS defence.
- **Limit batching / aliasing** and apply rate limiting that counts operations, not just HTTP requests, so a batch can't smuggle a thousand logins into one call.
- **Authorize per resolver/field**, using the authenticated identity — never assume reaching a field through the graph implies permission. This is where most real GraphQL bugs live.
- **Validate and parameterise resolver arguments** against injection, exactly as for REST.
- Add timeouts and pagination on list fields so relationship traversal can't return unbounded data.

## Pitfalls

- **Leaving introspection on in prod.** Convenient in dev, a free schema map in prod.
- **Rate limiting HTTP requests only.** Batching/aliasing does many operations per request and walks straight through it. Count operations.
- **Endpoint-level authz thinking.** GraphQL has one endpoint; authorization must be per field/resolver, or the graph becomes a traversal path to everything.
- **Forgetting the resolvers hit real databases.** All the injection classes still apply through GraphQL arguments.

## References

- OWASP API Security Top 10 (API4 resource consumption, API1/API3 authorization)
- OWASP GraphQL Cheat Sheet
- graphql-cop and GraphQL security testing tooling
- CWE-770, CWE-639
