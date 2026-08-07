---
name: dependency-confusion
domain: 09-software-supply-chain-security
description: Use when protecting against dependency confusion attacks — where an attacker publishes a public package matching your internal package name and your build pulls theirs instead.
difficulty: intermediate
tags: [supply-chain, dependency-confusion, packages, namespace, registries]
tools: [npm, pip, artifactory]
---

## Purpose

Dependency confusion is a supply-chain attack that exploits how package managers resolve names: if you use an internal package `mycompany-utils` and an attacker publishes a `mycompany-utils` to the *public* registry with a higher version number, your build may pull the attacker's public package instead of your private one — running their code in your build and production. This skill covers understanding and preventing dependency confusion, a widely-exploitable class discovered to affect major companies.

## When to use it

Any organisation using internal/private packages alongside public registries (npm, PyPI, etc.) — which is most. It's a high-impact, easily-overlooked attack because it exploits default package-manager behaviour, not a bug in your code.

## How the attack works

Package managers often check multiple sources (your private registry and the public one) and, by default, may pick the *highest version* regardless of source. So:
1. An attacker learns an internal package name (often leaked in a public repo, a `package.json`, or an error message).
2. They publish a package with that exact name to the public registry, with a very high version number.
3. Your build resolves the dependency, sees the attacker's higher-versioned public package, and installs *it* instead of your internal one — executing attacker code at install time (install scripts run) and in your application.

## Procedure

1. **Understand your exposure.** Do you use internal packages? Are their names discoverable (in public repos, client-side bundles, error messages)? Does your package manager check public registries for names you intend to be private? If yes to these, you're exposed.
2. **Claim your namespaces on public registries — the simplest defence.** Register your internal package names (or a scope/namespace prefix) on the public registry yourself, so an attacker can't publish under them. Using scoped packages (`@mycompany/utils`) with a reserved scope prevents the public registry from serving an attacker's package under your scope.
3. **Configure the package manager to resolve internal packages only from your private registry.** Explicitly scope which packages come from where (npm `.npmrc` scope-to-registry mapping, pip index configuration), so internal names are *never* resolved from the public registry. This removes the confusion at the resolution level.
4. **Use a single, controlled registry (a proxy/virtual registry).** Front both public and private packages through one managed registry (Artifactory, Nexus) that you control, with rules preventing public packages from shadowing internal names. This centralises and controls resolution.
5. **Pin and verify.** Lockfiles with integrity hashes (the lockfile-integrity skill) ensure you install the exact package you expect; verify sources.
6. **Don't leak internal package names.** Keep `package.json`/requirements with internal names out of public repos and client bundles where feasible — name discovery is the attacker's first step.
7. **Test your exposure.** Check whether your build would pull a public package over an internal one of the same name (safely) — if it would, the confusion path is open.

## Cheatsheet

```
attack: internal pkg `mycompany-utils` + attacker publishes SAME name to PUBLIC registry
  at a HIGHER version -> your build pulls THEIRS (highest-version-wins default) -> attacker code runs

how: package managers check multiple sources + default to highest version regardless of source
  attacker: learn internal name (leaked) -> publish public w/ huge version -> your build installs it

defend
  CLAIM your namespaces on public registries (or reserved SCOPE: @mycompany/utils) — simplest
  SCOPE resolution: internal names resolve ONLY from private registry (.npmrc / pip index config)
  SINGLE controlled registry (Artifactory/Nexus proxy) — rules stop public shadowing internal
  PIN + verify (lockfile integrity hashes)
  DON'T LEAK internal package names (public repos, client bundles, errors) — name = attacker's step 1
  TEST: would your build pull a public pkg over an internal same-name one?
```

## Reading the exposure

- **Internal packages resolvable from the public registry** = the confusion path is open; an attacker who learns the name and publishes a higher version gets their code into your build. The core vulnerability, and it exploits default behaviour, not a bug.
- **Unclaimed internal namespaces on public registries** = an attacker can publish under your internal names; claiming them (or a reserved scope) is the simplest, most effective defence.
- **Package manager checking public registries for private names** = the mechanism of the attack; scope resolution so internal names only come from your private registry, and the confusion can't happen.
- **Internal package names leaked** (in a public `package.json`, a client bundle, an error) = the attacker's first step accomplished for them; keep internal names private where feasible.
- **No single controlled registry** = resolution is harder to govern; a proxy/virtual registry with anti-shadowing rules centralises control.
- **Scoped/claimed namespaces, private-only internal resolution, controlled registry, integrity-pinned** = the confusion attack is closed.

## Pitfalls

- **Relying on package-manager defaults.** The default highest-version-wins-across-sources behaviour is exactly what the attack exploits; you must explicitly scope resolution or claim namespaces. Defaults leave you exposed.
- **Not claiming internal namespaces publicly.** Leaving your internal names unregistered on the public registry lets an attacker publish under them; claim them or use reserved scopes.
- **Leaking internal package names.** Name discovery is the attacker's first step; internal names in public repos, client bundles, or error messages hand it to them.
- **Mixed public/private resolution without scoping.** If internal names can resolve from public, the confusion path is open; scope which packages come from where.
- **Assuming it's a niche issue.** Dependency confusion affected major companies precisely because it exploits universal default behaviour; treat it as a real, common exposure.

## References

- Alex Birsan's dependency confusion research (the disclosure that named the attack)
- npm scopes / `.npmrc`, pip index configuration, and private-registry documentation
- Artifactory/Nexus virtual-registry configuration
- The lockfile-integrity and package-repo-hardening skills
