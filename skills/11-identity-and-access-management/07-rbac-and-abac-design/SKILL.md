---
name: rbac-and-abac-design
domain: 11-identity-and-access-management
description: Use when designing an authorization model — choosing between role-based and attribute-based access control, and structuring permissions that stay correct as the system grows.
difficulty: intermediate
tags: [iam, authorization, rbac, abac, access-control, design]
tools: []
---

## Purpose

Authentication proves who you are; authorization decides what you can do — and it's where most access-control bugs are born, because permission models that start simple rot into a tangle nobody can reason about. This skill covers choosing an authorization model (RBAC, ABAC, or a blend) and structuring it so access decisions stay correct and auditable as the system scales.

## When to use it

Designing a new system's permissions, or refactoring a permission model that's become unmanageable (roles multiplying, one-off exceptions everywhere, nobody able to answer "who can do X"). It's the design counterpart to the access-control *testing* skills in the web/API domains.

## The models

- **RBAC (role-based)** — permissions attach to roles, users get roles. Simple, auditable, and the right default for most systems: "editors can publish", "admins can delete". Struggles when access depends on context beyond the role (this record, this time, this location).
- **ABAC (attribute-based)** — decisions are computed from attributes of the user, the resource, the action, and the environment ("a manager can approve expenses *in their own department* under $X"). Powerful and fine-grained, but more complex to reason about and audit.
- **ReBAC (relationship-based)** — access follows relationships ("can edit documents they own or that are shared with them"); common in collaborative apps.
- **In practice, most systems blend them**: RBAC for the coarse structure, attributes for the contextual conditions. Start with RBAC and add attribute conditions where the role alone can't decide.

## Procedure

1. **Enumerate the resources and actions** first — what can be done to what. This is the vocabulary of your permissions; get it explicit before assigning anyone anything.
2. **Choose the model by how access actually depends on things.** If access is well-described by job function alone, RBAC. If it genuinely depends on context (ownership, department, amount, time), you need attribute conditions — but don't reach for full ABAC complexity if a handful of roles would do.
3. **Design roles around function, not individuals** — a role should map to a job/responsibility, granting the least privilege that job needs. Avoid per-person roles and "temporary" roles that never die.
4. **Keep the permission model centralized and declarative** — a single place that expresses the policy (a policy engine, a permissions table), not authorization logic scattered and duplicated across the codebase where it drifts and gets forgotten (the root of most access-control bugs).
5. **Default deny.** Access not explicitly granted is denied. A model that defaults to allow, or where a missing check means access, fails open.
6. **Plan for review.** Design so you can answer "who can do X?" and "what can this user do?" — periodic access review and least-privilege audits depend on the model being legible. If nobody can answer those, the model is already broken.
7. **Handle escalation and separation of duties** where needed — step-up for sensitive actions, and splitting conflicting permissions (the person who requests can't also approve).

## Cheatsheet

```
pick the model by how access DEPENDS
  role/function alone decides       -> RBAC (default, simplest, auditable)
  depends on ownership/relationship -> ReBAC (collaborative apps)
  depends on context (dept, amount, -> ABAC / attribute conditions on RBAC
    time, location)
  most real systems = RBAC skeleton + attribute conditions where needed

design rules
  [ ] enumerate resources x actions first (the permission vocabulary)
  [ ] roles = job functions, least privilege, no per-person roles
  [ ] centralized + declarative policy (one place), not scattered checks
  [ ] DEFAULT DENY (missing grant = no access)
  [ ] legible: can you answer "who can do X?" / "what can user Y do?"
  [ ] separation of duties + step-up for sensitive actions

smell: roles multiplying, one-off exceptions, nobody can answer "who can X?"
```

## Reading a design

- **Authorization logic scattered across the codebase** = the model will drift and develop gaps; each ad-hoc check is a future access-control bug. Centralize it.
- **A model that defaults to allow** (or where a forgotten check grants access) = fails open; one omission becomes an exposure. Default deny.
- **Roles multiplying toward one-per-user** = RBAC misapplied; either the roles aren't really functional, or you actually needed attribute conditions. Rethink.
- **Nobody can answer "who can do X?"** = the model isn't auditable, which means least-privilege review is impossible and privilege creep is invisible. Legibility is a security property here.
- **A clean role structure with attribute conditions only where context demands** = the pragmatic sweet spot; note it as the target.

## Pitfalls

- **Scattered, duplicated authorization checks.** The single biggest source of access-control bugs — logic drifts, one path forgets the check. Centralize the policy.
- **Reaching for full ABAC when RBAC would do.** Fine-grained attribute policies are powerful and hard to audit; use them where context genuinely decides, not everywhere.
- **Per-person and "temporary" roles.** They accumulate, defeat auditability, and become privilege creep. Roles map to functions.
- **Fail-open defaults.** Access must be explicitly granted; a missing decision must deny.
- **Ignoring auditability.** If you can't enumerate who has access to what, you can't do least-privilege review — and privilege creep goes unchecked.

## References

- NIST RBAC model (INCITS 359) and NIST SP 800-162 (ABAC)
- OWASP Authorization Cheat Sheet
- Google Zanzibar (ReBAC) as a reference for relationship-based models
- CWE-284, CWE-269 (improper access control / privilege management)
