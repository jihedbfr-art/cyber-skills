---
name: firewall-rule-review
domain: 02-network-security
description: Use when auditing a firewall rule set for overly-permissive, shadowed, or stale rules — the accumulated cruft that quietly widens what's allowed through.
difficulty: intermediate
tags: [network, firewall, rules, audit, hardening]
tools: [nipper, pfsense]
---

## Purpose

Firewall rule sets rot. Rules get added for a project and never removed, "temporary" any-any rules become permanent, and new rules quietly shadow old ones. Over time the effective policy allows far more than anyone intended. This skill covers reviewing a rule set to find the permissive, redundant, shadowed, and stale rules, and tightening it back toward least access.

## When to use it

Periodic firewall audits, after an assessment, during a migration, or when nobody can confidently answer "what does this firewall actually allow?". It pairs with the segmentation skill — the firewall is often where segmentation boundaries are enforced (or quietly aren't).

## Procedure

1. **Get the full rule set and read it in order.** Firewalls evaluate rules top-down, first match wins — so order matters as much as the rules themselves. Export the config for offline analysis.
2. **Hunt overly-permissive rules** — the biggest risk. `any` in source, destination, service, or port is where exposure hides. An `any-any allow` is effectively no firewall for that path:
   ```
   any source -> any dest -> any service : ALLOW     -> the rule that undoes the firewall
   any -> internal sensitive host : ALLOW            -> exposure
   ```
3. **Find shadowed rules** — a broad rule earlier in the list that makes a later, more specific rule never fire. Shadowed rules signal confused policy and sometimes hide that a "deny" is never reached because an "allow" above it matches first.
4. **Find redundant/duplicate rules** that add clutter and make the set harder to reason about — cleanup that reduces the chance of a mistake.
5. **Find stale rules** — allowing traffic to hosts that no longer exist, for projects long finished. Cross-reference with current assets; a rule allowing access to a decommissioned server is dead weight and sometimes a re-exposure risk if the IP is reused.
6. **Check the default action** — the rule set should end in an explicit deny-all with logging. If the default is allow, or the final deny isn't logged, that's a finding.
7. **Automate where possible** — tools like Nipper analyse configs for these issues across vendors; use them to scale the review, then verify by hand.

## Cheatsheet

```
review order: rules evaluate TOP-DOWN, first match wins — read in sequence

find these
  overly permissive   any src/dst/service, esp. any-any allow  (biggest risk)
  to sensitive hosts   allow from broad sources to DB/DC/admin
  shadowed            broad rule above makes a specific rule never fire
  redundant           duplicate/overlapping rules -> clutter
  stale               allow to decommissioned hosts / finished projects
  default action      must end in explicit DENY-ALL + logging

questions per rule
  is the source as narrow as possible?  the destination?  the port/service?
  is this rule still needed?  who owns it?  is it logged?
```

## Reading the review

- **An `any-any allow` (or any-any on a sensitive path)** = effectively no firewall there; the top-severity finding. Every path it covers is open.
- **Broad source allowed to a sensitive host** (DB, DC, admin interface) = a direct exposure; tighten the source to the minimum, or move the host behind segmentation.
- **Shadowed rules** = the effective policy differs from what the admin thinks; a later deny that never fires, or a specific allow masked by a broad one. Reorder or remove to make intent match reality.
- **Stale rules to dead hosts** = clutter now, re-exposure risk if the IP gets reused. Remove them.
- **No final deny-all, or deny not logged** = traffic may fall through to a permissive default, and denied attempts go unseen. Add explicit logged deny-all.
- **A lean, ordered, least-privilege set ending in logged deny-all** = the good state.

## The fix

- **Remove or narrow overly-permissive rules** — replace `any` with the specific sources, destinations, and ports actually needed. Least access, per rule.
- **Reorder to eliminate shadowing** and remove redundant rules so the effective policy is legible and matches intent.
- **Prune stale rules** by cross-referencing current assets; retire rules for hosts and projects that are gone.
- **End with an explicit deny-all and log it**, so nothing falls through to a permissive default and denied traffic is visible.
- **Log allowed sensitive flows too**, and establish a change process (owner, review date per rule) so the set doesn't rot again.
- **Re-review periodically** — rule sets accumulate cruft continuously.

## Pitfalls

- **Missing shadowed rules.** The written policy and the effective policy diverge when a broad rule sits above a specific one; you have to read in evaluation order to catch it.
- **Leaving "temporary" any-any rules.** They're the most dangerous and the most likely to be forgotten. Hunt them specifically.
- **Cleaning rules without knowing who owns them.** Removing a rule that's still needed causes an outage; removing one nobody can justify is the win. Track ownership.
- **No logging on deny.** Without it, you can't see attacks or troubleshoot — and you lose a detection source.
- **One-time audit.** Rule sets rot continuously; schedule the review.

## References

- NIST SP 800-41 (Guidelines on Firewalls and Firewall Policy)
- Nipper / vendor config-audit tooling
- CIS benchmarks for firewall/router configuration
- The network-segmentation skill (firewalls enforce segment boundaries)
