---
format: "v2"
name: "grpc-security"
title: "Grpc Security"
title_fr: "Sécurité gRPC"
description: "Use when securing or testing a gRPC API — authentication, input validation, and the differences from REST that change how you attack and defend it."
description_fr: "À utiliser pour sécuriser ou tester une API gRPC — authentification, validation des entrées, et les différences avec REST qui changent la façon de l'attaquer et de la défendre."
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

gRPC is HTTP/2 plus protobuf, used heavily for service-to-service and increasingly for client-facing APIs. The security fundamentals are the same as any API — authenticate, authorize, validate — but the binary framing and the "internal by default" assumption change how issues show up and get missed. This skill covers the gRPC-specific angles.

### When to use it

Any gRPC service, especially one exposed beyond a trusted network. gRPC's reputation as an "internal" protocol means auth and validation are often weaker than on a public REST API — which is exactly why it's worth testing.

### Procedure

1. **Discover the interface.** If server reflection is enabled, you can enumerate services and methods without the `.proto` files — useful for testing, a disclosure to consider disabling in production:
   ```
   grpcurl -plaintext api.tld:443 list
   grpcurl -plaintext api.tld:443 describe some.Service
   ```
2. **Check transport security.** gRPC should run over TLS. A `-plaintext` connection succeeding on an exposed endpoint means traffic is unencrypted — a finding.
3. **Test authentication.** gRPC auth is usually a token in call metadata (like an HTTP header). Test the same failures as REST: does a method accept a missing/invalid/expired token? Are all methods protected, or only some?
   ```
   grpcurl -H "authorization: Bearer <token>" api.tld:443 some.Service/Method
   ```
4. **Test authorization per method and per object.** BOLA applies: can you call a method with another user's object ID? Enumerate the methods (reflection helps) and check each enforces access, not just authentication.
5. **Test input validation.** protobuf gives you typed fields, which stops some malformed input — but it does **not** validate business constraints or stop injection in the values. The resolvers behind gRPC hit databases; the usual injection tests apply to string fields.
6. **Check resource limits.** Large messages, streaming abuse, and unbounded requests are DoS vectors — confirm max message sizes and streaming limits are set.

### Cheatsheet

```bash
grpcurl -plaintext host:port list
grpcurl -plaintext host:port describe pkg.Service

grpcurl -H "authorization: Bearer $T" -d '{"id":"123"}' host:port pkg.Service/Get
grpcurl -d '{"id":"123"}' host:port pkg.Service/Get      # no token -> should fail

grpcurl -H "authorization: Bearer $T" -d '{"id":"<other>"}' host:port pkg.Service/Get

grpcurl -plaintext host:port list      # succeeds? traffic may be unencrypted
```

### Reading the output

- **`-plaintext` working on an exposed endpoint** = no/negotiable TLS; traffic is interceptable. Finding.
- **A method answering with no or invalid token** = broken authentication, same as REST — often worse on gRPC because it was assumed internal.
- **Reaching another user's object via a method call** = BOLA over gRPC; the binary framing doesn't make it any less exploitable.
- **Reflection exposing the full service map** in production = information disclosure; convenient for you as a tester, worth disabling for the defender.
- **Injection landing in a string field** = the protobuf type system validated the *shape*, not the *content* — the resolver is still vulnerable.

### The fix

- **Require TLS** (mTLS for service-to-service) — disable plaintext on anything exposed. mTLS also gives you strong service identity.
- **Authenticate every method** via call credentials/metadata, validated server-side; don't assume "internal" means safe.
- **Authorize per method and per object**, using the authenticated identity — a gateway/interceptor is a good place to enforce authentication uniformly, but object-level checks stay in the service.
- **Validate field content**, not just protobuf types — apply the injection and business-rule checks the type system doesn't. Use a validation layer (e.g. protovalidate/buf) for constraints.
- **Set message size and streaming limits** to bound resource use.
- **Disable server reflection in production** unless you specifically need it.

### Pitfalls

- **Assuming "internal" equals secure.** gRPC's service-mesh heritage leads teams to skip auth; exposed gRPC needs the same rigor as public REST.
- **Trusting protobuf as input validation.** It checks types, not values or business rules — injection and invalid-state bugs pass straight through.
- **Plaintext in production.** Easy to leave on from local dev; it exposes all traffic.
- **Authenticating but not authorizing per object.** BOLA is as common on gRPC as REST; typed IDs are still guessable/enumerable.

### References

- gRPC authentication and TLS documentation
- OWASP API Security Top 10 (applies to gRPC)
- buf / protovalidate for input constraints
- CWE-287, CWE-639

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.