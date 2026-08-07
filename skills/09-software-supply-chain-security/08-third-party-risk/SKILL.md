---
name: third-party-risk
domain: 09-software-supply-chain-security
description: Use when assessing the security risk of a third-party component, library, or vendor before depending on it — evaluating what you're taking on when you adopt someone else's code or service.
difficulty: intermediate
tags: [supply-chain, third-party, vendor-risk, dependencies, assessment]
tools: [scorecard, deps.dev]
---

## Purpose

Every dependency and vendor you adopt is a piece of your attack surface you don't control. A poorly-maintained library, an abandoned project, or a vendor with weak security becomes your risk the moment you depend on it. Third-party risk assessment is evaluating that risk *before* you take it on — so you make an informed decision rather than inheriting problems. This skill covers assessing the security of third-party components and vendors, the "should we depend on this" question.

## When to use it

Before adopting a significant new dependency, library, or vendor, and periodically re-assessing critical ones. It's a decision-support skill — the point is to make dependency and vendor choices deliberately, weighing the security risk, rather than adopting whatever's convenient.

## Procedure

1. **Assess the health and maintenance of a dependency.** An unmaintained or abandoned dependency won't get security fixes — a serious latent risk. Check: is it actively maintained (recent commits, releases)? How many maintainers (a single-maintainer critical dependency is a bus-factor and takeover risk)? Is it widely used and reviewed? How responsive are they to security issues? Tools like OpenSSF Scorecard and deps.dev automate much of this.
2. **Assess the security posture of the project.** Does it follow security practices — signed releases, a security policy, dependency hygiene, a track record of handling vulnerabilities well? A project with good security practices is a lower-risk dependency; one that's careless with its own security is careless with yours.
3. **Consider the dependency's own dependencies.** Adopting a component pulls in its transitive dependency tree — you inherit all of it. A component with a bloated or risky dependency tree brings that risk along. Assess the whole tree, not just the top-level package.
4. **For vendors/services, assess their security.** A SaaS vendor or service you integrate becomes part of your supply chain; assess their security posture (certifications like SOC 2, their track record, their access to your data and systems). A vendor breach can become your breach (the ties to GRC third-party-risk-management).
5. **Weigh necessity against risk.** The best way to reduce a dependency's risk is sometimes to not take it on — do you actually need this dependency, or could you avoid it? Every dependency is permanent risk surface; adopting fewer, higher-quality ones beats pulling in many.
6. **Make it a deliberate decision with a record.** Adopting a significant dependency/vendor should be a considered choice with the risk assessment recorded, not an ad-hoc `npm install`. For critical ones, this feeds the risk register (GRC domain).
7. **Re-assess periodically.** A well-maintained dependency can become abandoned; a vendor's posture can change. Critical third parties warrant periodic re-assessment, not a one-time check at adoption.

## Cheatsheet

```
every dependency/vendor = attack surface you DON'T control. assess BEFORE depending.

assess a DEPENDENCY
  HEALTH/maintenance: active? recent releases? # maintainers (single = bus-factor + takeover risk)?
    widely used/reviewed? responsive to security issues?  (OpenSSF Scorecard, deps.dev)
  SECURITY posture: signed releases? security policy? dependency hygiene? vuln track record?
  its OWN dependency TREE (you inherit ALL of it — assess the whole tree, not just top-level)

assess a VENDOR/service
  security posture (SOC 2 etc.), track record, their access to your data/systems
  vendor breach -> your breach (GRC third-party-risk-management)

WEIGH necessity vs risk: best risk reduction = don't take it on (do you NEED it?)
  fewer, higher-quality deps > many
DELIBERATE decision + RECORD (not ad-hoc npm install) ; critical -> risk register
RE-ASSESS periodically (maintained -> abandoned ; vendor posture changes)
```

## Reading the assessment

- **An unmaintained or abandoned dependency** = it won't get security fixes; a serious latent risk you inherit. A critical dependency that's no longer maintained is a strong reason to avoid or replace it.
- **A single-maintainer critical dependency** = a bus-factor risk and a takeover target (a compromised or coerced sole maintainer can push malicious code — this has happened). Weigh the concentration risk.
- **A dependency with a bloated or risky transitive tree** = you inherit all of it; the top-level package may be fine but its dependencies aren't. Assess the whole tree.
- **A vendor with weak security posture and access to your data** = their breach becomes yours; the vendor's security is your risk. Assess it before integrating (ties to GRC).
- **Adopting a dependency you don't really need** = permanent, avoidable risk surface; the strongest risk reduction is sometimes not taking it on. Question necessity.
- **A deliberate, recorded adoption decision weighing health, security, and necessity** = third-party risk managed rather than inherited blindly.

## Pitfalls

- **Adopting dependencies without assessment.** Every dependency is uncontrolled attack surface; adopting whatever's convenient inherits its problems (abandonment, poor security, risky tree). Assess before depending.
- **Ignoring maintenance health.** An unmaintained dependency won't get security fixes — a latent risk that surfaces when a CVE drops and there's no fix. Check activity and responsiveness.
- **Overlooking the transitive tree.** You inherit the whole dependency tree, not just the package you chose; a clean top-level with a risky tree is still risky.
- **Not assessing vendors' security.** A vendor with access to your data and weak security is a breach path into you; assess their posture (GRC).
- **Adopting unnecessary dependencies.** The best risk reduction is often not taking it on; question whether you need it. Fewer, higher-quality dependencies beat many.
- **One-time assessment.** Dependencies get abandoned and vendors change; re-assess critical third parties periodically.

## References

- OpenSSF Scorecard and deps.dev (automated dependency health/security assessment)
- The GRC third-party-risk-management skill (vendor risk) and this domain's other skills
- OpenSSF and NIST supply-chain risk-management guidance (NIST SP 800-161)
- The vulnerable-dependency-triage and malicious-package-response skills
