---
name: multi-account-guardrails
domain: 06-cloud-security
description: Use when structuring cloud across multiple accounts — using organization-wide guardrails (SCPs, landing zones) to enforce security that no single account can override.
difficulty: advanced
tags: [cloud, aws, organizations, scp, landing-zone, governance]
tools: [aws-cli]
---

## Purpose

Securing one cloud account is per-account work; securing an organisation of dozens is a governance problem. Multi-account guardrails set boundaries at the organisation level — controls that apply to every account and that individual account admins *cannot* turn off. This is how you stop the "one team disabled logging" or "someone made a public bucket" problem from being possible at all, rather than catching it after the fact. AWS Organizations/SCPs are the example; Azure Management Groups and GCP Organization Policies are the equivalents.

## When to use it

When your cloud footprint spans multiple accounts/subscriptions — separate accounts per team, environment, or workload (which is itself good practice for blast-radius isolation). It sits above the per-account cloud skills, enforcing a floor of security across all of them.

## The building blocks

- **Account separation** — using many accounts (per team/env/workload) to isolate blast radius; a compromise in one doesn't reach the others. This is the foundation multi-account security rests on.
- **Service Control Policies (SCPs)** — organisation-level guardrails that cap what any account can do, regardless of that account's IAM. An SCP denying an action means no principal in that account can perform it, even an account admin. This is the enforcement mechanism.
- **Landing zone** — a standardised, pre-secured account baseline (logging, guardrails, network, identity) applied to every new account, so accounts start secure rather than being hardened later.
- **Centralised logging and identity** — a dedicated logging account (per the CloudTrail skill) and central SSO, so audit trails and access are managed org-wide and out of workload accounts' reach.

## Procedure

1. **Separate accounts by blast radius** — team, environment (prod/dev), and sensitive workloads in their own accounts, so a compromise is contained. This structure is what makes the guardrails meaningful.
2. **Set preventive guardrails with SCPs** — encode the non-negotiables as org-level denies that account admins can't override. Common guardrails: deny disabling CloudTrail/Config, deny leaving the region set you operate in, deny making certain resources public, deny root-user actions, restrict which services can be used:
   ```
   aws organizations list-policies --filter SERVICE_CONTROL_POLICY
   aws organizations describe-policy --policy-id <id>
   ```
3. **Standardise account creation with a landing zone** (AWS Control Tower or an IaC baseline) so every new account arrives with logging on, guardrails attached, and a secure network/identity baseline — no manual hardening step to forget.
4. **Centralise logging and identity** — deliver all accounts' audit logs to a locked-down logging account, and manage human access through central SSO with least privilege, so workload accounts can't tamper with their own evidence or manage their own users.
5. **Layer detective controls** (org-wide CSPM/Config rules) on top of the preventive SCPs — guardrails stop the obvious, monitoring catches the rest across all accounts.
6. **Test the guardrails** — confirm an account admin genuinely cannot perform a denied action; an SCP you haven't verified is an assumption.

## Cheatsheet

```
structure: many accounts by blast radius (team / env / sensitive workload)

preventive (SCPs — account admins CANNOT override)
  deny disabling CloudTrail / Config           (protect logging)
  deny leaving approved regions
  deny making resources public (where policy demands)
  deny root-user actions
  restrict allowed services

standardise: landing zone (Control Tower / IaC) -> new accounts start secure
centralise:  logging account (locked down) + central SSO (least privilege)
detective:   org-wide CSPM/Config rules on top of SCPs
verify:      confirm an account admin truly can't do a denied action

commands
  aws organizations list-policies --filter SERVICE_CONTROL_POLICY
  aws organizations list-accounts / list-organizational-units-for-parent
```

## Reading the setup

- **A single account (or a few) holding everything** = no blast-radius isolation; one compromise reaches it all. Separate by team/env/workload first — guardrails matter less without this.
- **No SCPs / guardrails only advisory** = every control is optional at the account level; an account admin can disable logging or go public. Preventive SCPs are what make security non-negotiable.
- **New accounts hardened manually** = inconsistent and error-prone; the account that skipped a step is the one that gets breached. A landing zone makes secure the default.
- **Workload accounts managing their own logs/identity** = they can tamper with evidence and over-grant access. Centralise both out of their reach.
- **SCPs defined but never tested** = you're assuming the guardrail holds; verify an admin can't perform the denied action.

## The fix / best practice

- **Adopt a multi-account structure** with separation by blast radius as the foundation.
- **Enforce preventive SCPs** for the non-negotiables (protect logging, restrict regions/services, block root and unwanted public exposure) so no account can override them.
- **Use a landing zone** (Control Tower or IaC) so every account starts from a secure, guardrailed baseline automatically.
- **Centralise logging (dedicated account) and identity (SSO)** org-wide, keeping them out of workload accounts.
- **Add org-wide detective controls** (CSPM/Config) as the layer that catches what guardrails don't prevent.
- **Verify guardrails actually enforce**, and review them as the org evolves.

## Pitfalls

- **Guardrails that are only advisory.** If a control can be turned off by an account admin, it isn't a guardrail. Use SCPs for the things that must hold everywhere.
- **No account separation.** Guardrails on a monolithic account don't give blast-radius isolation; the multi-account structure is the point.
- **Manual account setup.** Hand-hardening each new account guarantees drift and gaps. Standardise with a landing zone.
- **Letting workload accounts own their logging/identity.** They can then tamper with evidence or over-grant. Centralise.
- **SCPs so broad they block legitimate work** (or so narrow they protect nothing) — tune them, and test both that they allow real work and deny the bad.

## References

- AWS Organizations / SCPs and AWS Control Tower documentation
- AWS Security Reference Architecture (multi-account patterns)
- Azure Management Groups / GCP Organization Policy equivalents
- CIS cloud foundations benchmarks
