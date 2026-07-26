---
name: containment-strategies
domain: 22-incident-response
description: Use once an incident is confirmed and scoped — deciding how to stop the spread without destroying evidence or tipping off the attacker prematurely.
difficulty: intermediate
tags: [incident-response, containment, isolation, evidence]
tools: []
---

## Purpose

Containment is the hinge of incident response: the point where you stop the bleeding. Done well, it limits damage while preserving what you need to investigate and recover. Done hastily, it destroys evidence, alerts the attacker to dig in deeper, or takes down more than the incident would have. This skill covers choosing a containment approach that fits the situation.

## When to use it

Right after triage confirms a real incident and you have a rough scope (the triage skill). Containment comes before eradication — you isolate first, then clean. It's a decision made under pressure, which is exactly why having the trade-offs straight in advance matters.

## The core tension

Every containment choice trades three things against each other:

- **Speed of stopping spread** vs **preserving evidence** — pulling power stops an attacker but destroys memory and volatile artifacts the forensics skill needs.
- **Containing visibly** vs **staying covert** — blocking the attacker loudly may cause them to burn what they have (deploy ransomware, wipe logs) before you're ready.
- **Scope of impact** — isolating a segment may disrupt the business more than the incident itself; sometimes the cure must be weighed against the disease.

## Procedure

1. **Decide short-term vs long-term containment.** Short-term = stop the immediate spread now (isolate a host, block an IP, disable an account). Long-term = the more durable holding measure while you prepare eradication (rebuild in a clean segment, apply a temporary fix). Usually you do short-term immediately, then move to long-term.
2. **Preserve before you cut, where feasible.** If the host will be investigated, capture volatile data (memory, network state) before isolating or powering off — pulling the plug throws away RAM. Balance this against how fast the threat is spreading; a live wiper doesn't wait for your memory capture.
3. **Isolate without destroying.** Prefer **network isolation** (cut the host off but leave it running) over powering down — it stops spread and C2 while preserving the live state. Disable rather than delete accounts.
4. **Think about the attacker's reaction.** If they have broad access, a visible block can trigger destructive action. Consider containing everything you can in one coordinated move rather than tipping them off piecemeal — decide with the incident lead.
5. **Scope the containment to the incident.** Isolate what's affected plus a safety margin, not the whole network by reflex — but don't under-contain and let it spread past your line either.
6. **Document every action with timestamps** — what you isolated, when, and why. Recovery and the postmortem depend on it.

## Cheatsheet

```
containment order of operations
  1. short-term: stop spread NOW (isolate host / block IP / disable account)
  2. preserve:   capture volatile evidence before power-off (if time allows)
  3. long-term:  durable holding measure while preparing eradication

isolate WITHOUT destroying
  network-isolate a host (keep it running)   > power it off (loses RAM)
  disable an account                          > delete it (loses trail)
  block C2 at the firewall                     > wipe the host immediately

weigh before acting
  spread speed vs evidence preservation
  visible block vs attacker retaliation (ransomware/wipe)
  containment blast radius vs the incident's own impact

always: timestamp and log every containment action
```

## Reading the situation

- **Active, fast spread (a worm, live ransomware)** shifts the balance toward speed — isolate aggressively now, accept some evidence loss. A live wiper won't wait.
- **A quiet, established foothold** favours careful, coordinated containment — moving loudly may push the attacker to act destructively. Plan the cut with the lead.
- **A host you'll need to investigate** argues for network isolation over power-off, to keep memory and live state intact.
- **Containment that would disrupt critical business more than the incident** needs an explicit risk decision by the incident owner — sometimes you tolerate a contained threat briefly to avoid a self-inflicted outage.
- **"We're not sure of full scope"** means contain conservatively but keep hunting — under-containment lets it spread, over-containment causes its own damage.

## Pitfalls

- **Powering off a host you needed to investigate.** RAM and volatile artifacts are gone. Network-isolate instead where the situation allows.
- **Tipping off the attacker piecemeal.** Blocking one thing at a time on a broadly-compromised environment invites retaliation. Coordinate the cut.
- **Deleting instead of disabling.** A deleted account/host destroys the very trail the investigation follows. Disable and preserve.
- **Containing the whole business by reflex.** Over-isolation can outweigh the incident's own harm; scope to the affected area plus margin, decided with the owner.
- **No documentation.** Undocumented containment actions confuse recovery and the postmortem. Timestamp everything.

## References

- NIST SP 800-61r2 (Containment, Eradication, and Recovery)
- SANS Incident Handler's Handbook
- MITRE ATT&CK (to anticipate attacker reaction during containment)
