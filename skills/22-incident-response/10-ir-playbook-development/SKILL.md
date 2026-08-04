---
name: ir-playbook-development
domain: 22-incident-response
description: Use when writing incident response playbooks before you need them — turning a scenario into concrete, tested steps so the response isn't improvised at 3am.
difficulty: intermediate
tags: [incident-response, playbook, runbook, preparation, process]
tools: []
---

## Purpose

The worst time to figure out your response is during the incident. A playbook is the pre-written, agreed procedure for a specific scenario — who does what, in what order, with which decisions escalated to whom. Good playbooks turn a panicked scramble into a calm sequence. This skill covers writing playbooks that actually get used, rather than binders nobody opens.

## When to use it

During the preparation phase — before incidents, not after. Build playbooks for the scenarios most likely to hit you or most damaging if they do: ransomware, account compromise, data breach, DDoS, a cloud key leak. Each of the response skills in this domain is the raw material; a playbook assembles them into a scenario-specific sequence.

## Procedure

1. **Pick scenarios by likelihood and impact**, not by what's easy to write. Start with the handful you're most likely to face (phishing-led account compromise, ransomware) — a playbook for an exotic threat you'll never see is wasted effort while the common case is unwritten.
2. **Structure each playbook around the IR lifecycle** so nothing is skipped under pressure: detection/triage → containment → eradication → recovery → post-incident. Reference the detailed skills rather than duplicating them.
3. **Make every step concrete and assignable.** "Contain the host" is not a step; "the on-call responder network-isolates the host via [tool], preserving power" is. Someone reading it cold at 3am should be able to act without interpretation.
4. **Name roles, not people.** Assign each step to a role (incident lead, comms lead, on-call engineer) so the playbook survives staff changes and works whoever is on shift. List who fills each role separately.
5. **Mark the decision points and escalations explicitly** — where the responder must stop and get a decision (declare a formal incident, involve legal, decide on ransom, take production offline). Say who owns each decision so it isn't made at the wrong level.
6. **Include the contacts and prerequisites inline** — emergency contacts, out-of-band comms channel, where the logs and backups are, which tools and access are needed. A playbook that sends people hunting for a phone number mid-incident has already failed.
7. **Test it.** Run the playbook in a tabletop exercise against a realistic scenario. Testing is what separates a real playbook from a document — it exposes missing steps, wrong assumptions, and access nobody actually has. Update from what the exercise reveals.
8. **Keep them current.** Review after real incidents (feed the postmortem back in) and on a schedule, because environments and tools drift.

## Cheatsheet

```
playbook structure (per scenario)
  scenario + trigger      what this playbook is for; how it's recognised
  roles                   incident lead / comms / on-call (roles, not names)
  detection & triage      confirm real, assign severity (-> triage skill)
  containment             concrete, assignable steps (-> containment skill)
  eradication & recovery  (-> eradication skill)
  decision points         declare? legal? ransom? prod offline? — WHO decides
  communication           who/when/channel (-> comms skill)
  contacts & prereqs      emergency contacts, OOB channel, logs/backups location
  post-incident           postmortem (-> postmortem skill)

quality tests
  [ ] can a responder act on each step COLD, no interpretation?
  [ ] roles not names? contacts inline?
  [ ] decision points marked + owned?
  [ ] tested in a tabletop, then updated?

pick scenarios by: likelihood x impact (ransomware, account compromise,
data breach, key leak, DDoS) — common case first, not exotic threats.
```

## Reading a draft playbook

- **Vague steps** ("investigate", "contain") = it'll be improvised anyway. Rewrite each as a concrete action a specific role can execute cold.
- **Steps assigned to named individuals** = brittle; the playbook breaks when that person leaves or is off-shift. Use roles.
- **No marked decision points** = responders either overstep their authority or freeze. Mark where to escalate and who decides.
- **Contacts/prerequisites missing** = mid-incident scavenger hunt. Put them inline.
- **Never tested** = not a playbook yet, just a wish. A tabletop exercise is what proves it works and surfaces the gaps.

## Pitfalls

- **Writing for exotic threats while the common case is unwritten.** Prioritise by likelihood × impact; the phishing-to-account-compromise playbook matters more than the nation-state one.
- **Vague, unassignable steps.** If it needs interpretation at 3am, it's not a playbook. Concrete + role-assigned.
- **Naming people instead of roles.** Staff change; the incident doesn't wait. Roles, with a separate roster.
- **Never testing.** Untested playbooks fail on first contact — missing access, wrong assumptions, steps in the wrong order. Tabletop them.
- **Set-and-forget.** Tools and environments drift; a stale playbook misleads. Review after incidents and on a cadence.

## References

- NIST SP 800-61r2 (preparation phase)
- CISA incident response playbooks (federal templates, adaptable)
- SANS Incident Handler's Handbook
- MITRE ATT&CK (to ground scenario steps in real attacker behaviour)
