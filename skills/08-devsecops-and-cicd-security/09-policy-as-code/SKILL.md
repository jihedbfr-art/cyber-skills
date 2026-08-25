---
format: "v2"
name: "policy-as-code"
title: "Policy As Code"
title_fr: "Politique as code"
description: "Use when codifying security gates so they can't be skipped — expressing policy as versioned, testable code enforced automatically in the pipeline instead of relying on manual review."
description_fr: "À utiliser pour coder les contrôles de sécurité afin qu'ils ne puissent plus être contournés — en exprimant la politique sous forme de code versionné et testable, appliqué automatiquement par le pipeline plutôt que confié à une revue manuelle."
domain: "08-devsecops-and-cicd-security"
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

Security policies enforced by manual review or documentation get skipped under deadline pressure — the whole point of DevSecOps is that security gates run automatically and consistently. Policy as code expresses your security and compliance rules as versioned, testable code that the pipeline enforces on every change, so a policy can't be forgotten, bypassed, or applied inconsistently. This skill covers codifying policy so the gates are automatic and reliable, the discipline that ties the DevSecOps controls together.

### When to use it

Maturing a DevSecOps practice from "we have guidelines" to "the pipeline enforces them". It's what makes SAST, dependency scanning, IaC scanning, and artifact verification into hard, consistent gates rather than optional steps, and it applies the same discipline to custom organisational policy.

### Procedure

1. **Codify the policies that should be enforced, not documented.** Any security or compliance rule that must hold — "no deploy without passing scans", "no public storage", "images must be signed", "no secrets in code", "required approvals for prod" — becomes code the pipeline checks automatically. Documentation is advisory; code is enforced.
2. **Use a policy engine.** Open Policy Agent (OPA) with Rego, Conftest (OPA for config files), or platform tools (Sentinel for Terraform) let you write policies as code that evaluate pipeline artifacts, configs, and deployments. The engine runs the policy on every change.
3. **Version, review, and test the policies — they're code.** Store policies in git, review changes via pull request, and *test* them (do they reject what they should and allow what they shouldn't?). A policy that blocks legitimate work causes outages; one that's too loose provides false assurance. Testing policies is as important as testing the code they govern.
4. **Enforce consistently across the pipeline.** The value is consistency — the same policies apply to every change, every team, every deployment, with no exceptions under deadline pressure. Manual review is inconsistent (different reviewers, skipped under pressure); codified policy is uniform.
5. **Provide clear feedback on violations.** When a policy blocks something, the developer needs to know *which* policy, *why*, and *how to comply* — in their workflow. A cryptic policy failure causes frustration and workarounds; clear guidance drives compliance.
6. **Handle exceptions as code too.** Real exceptions exist (a documented, approved deviation); handle them explicitly in the policy (an allowlist with justification) rather than by disabling the gate. An exception should be a deliberate, recorded decision, not a bypassed control.
7. **Roll out carefully.** Like any enforcement, start in warn/audit mode, find what breaks, fix it, then enforce — so codifying policy doesn't suddenly block everyone.

### Cheatsheet

```
manual/documented policy = SKIPPED under deadline pressure. codify -> automatic + consistent + unskippable.

codify what must HOLD (not just be documented)
  no deploy w/o passing scans | no public storage | images must be signed
  | no secrets in code | required prod approvals

engine: OPA/Rego | Conftest (config files) | Sentinel (Terraform)
  -> runs policy on every change (artifacts, configs, deployments)

policies are CODE: git + review + TEST (reject the bad, allow the good)
  too strict = outage ; too loose = false assurance ; testing policy = as important as code
ENFORCE CONSISTENTLY (same policy, every change/team/deploy, no deadline exceptions)
  vs manual review = inconsistent + skipped under pressure
clear VIOLATION feedback (which policy / why / how to comply — in workflow)
EXCEPTIONS as code (allowlist + justification), not a disabled gate — deliberate + recorded
roll out warn/audit -> fix -> enforce
```

### Reading the practice

- **Security policy enforced by manual review or documentation** = inconsistent and skipped under pressure; different reviewers apply it differently, and deadline pressure bypasses it. Codifying it makes the gate automatic and uniform — the core DevSecOps value.
- **Policies as versioned, tested code** = reliable gates; because they're tested, they reject what they should and allow legitimate work. Untested policies cause outages (too strict) or false assurance (too loose).
- **Consistent enforcement across every change and team** = the point; codified policy applies uniformly where manual review is patchy. No exceptions under deadline pressure.
- **Cryptic policy failures** = developer frustration and workarounds; violations must explain which policy, why, and how to comply, in the workflow. Clear feedback drives compliance.
- **Exceptions handled by disabling the gate** = a bypassed control; real exceptions should be explicit, justified allowlist entries — a recorded decision, not a hole.
- **Codified, tested, consistently-enforced policy with clear feedback and explicit exceptions** = DevSecOps gates that actually hold, tying the other controls into unskippable enforcement.

### Pitfalls

- **Relying on manual review or documentation.** It's inconsistent and skipped under deadline pressure; the whole point is automatic, uniform enforcement. Codify the policy.
- **Untested policies.** A policy that blocks legitimate work causes outages; one too loose gives false assurance. Test policies like the code they govern.
- **Cryptic violation messages.** Developers can't comply with a failure they don't understand, so they work around it; give clear which/why/how feedback in the workflow.
- **Exceptions by disabling the gate.** That's a bypassed control; handle exceptions as explicit, justified, recorded allowlist entries.
- **Enforcing without a rollout.** Codifying and immediately enforcing untested policy blocks everyone; use warn/audit mode first.
- **Codifying everything rigidly.** Some things genuinely need human judgement; codify the clear rules, and don't force nuanced decisions into brittle policy.

### References

- Open Policy Agent (OPA/Rego), Conftest, and HashiCorp Sentinel documentation
- The Kubernetes admission-control skill (policy-as-code for clusters) and the other DevSecOps gates
- OWASP CI/CD Security and DevSecOps guidelines
- The GRC domain (compliance-as-code overlaps)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.