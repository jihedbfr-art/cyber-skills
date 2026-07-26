---
name: ransomware-response
domain: 22-incident-response
description: Use when responding to a ransomware incident — the specific playbook for containment, the pay/don't-pay reality, recovery from backups, and what makes it different from other incidents.
difficulty: advanced
tags: [incident-response, ransomware, recovery, backups, extortion]
tools: []
---

## Purpose

Ransomware is the incident everyone fears and the one where the first hour decides the outcome. It combines active destruction (encryption in progress), extortion, and often data theft — so the response differs from a normal intrusion. This skill is the ransomware-specific playbook: contain the encryption, preserve options, and recover, without making the panic-driven mistakes that turn a bad day into a catastrophe.

## When to use it

The moment ransomware is suspected — encrypted files, ransom notes, mass file renames, or an alert on known ransomware behaviour. It builds on the general IR skills (triage, containment, evidence) with the specifics that ransomware demands.

## Procedure

1. **Contain the encryption immediately — this is the priority.** Ransomware spreads and encrypts actively, so every minute of delay is more data lost. **Network-isolate** affected hosts fast (disconnect from the network, keep them powered on to preserve memory and any keys in RAM). Isolate shared storage and disable the spreading mechanism (a compromised account, a scheduled task).
2. **Preserve evidence while containing.** Capture memory where feasible before anything reboots — encryption keys or the malware may be recoverable, and it aids attribution. Don't wipe hosts yet; you need them for scoping and possible decryption.
3. **Scope it.** Which hosts, which shares, which accounts? Identify the ransomware family (a note, file extension, or ID service can tell you) — it informs whether a free decryptor exists and whether data theft is part of this group's playbook.
4. **Check for data exfiltration.** Modern ransomware usually steals data before encrypting (double extortion). Whether data left the network changes this from an availability incident into a breach with notification obligations — investigate outbound transfer, don't assume encryption was the only harm.
5. **Assess recovery from backups — the real answer to the ransom.** Confirm you have backups, that they're **offline/immutable and not also encrypted**, and test-restore before wiping anything. Clean backups are what let you refuse to pay.
6. **The pay/don't-pay reality (not advice, decision factors):** paying is a business/legal/executive decision, never a technical one — surface the factors and escalate. Paying doesn't guarantee working decryption, funds criminal operations, may carry legal/sanctions exposure, and marks you as willing to pay. Even if paid, you still rebuild trust in every compromised system. Bring in legal, leadership, insurer, and often law enforcement before this is decided.
7. **Eradicate and recover from clean state.** Rebuild affected systems from known-good images/backups, reset credentials broadly (assume they're all compromised), close the entry vector, and restore in a controlled order. Don't just decrypt-in-place and move on — the attacker was in your environment.
8. **Communicate and report** per your plan — internal stakeholders, and external obligations (regulators, affected parties) especially if data was stolen.

## Cheatsheet

```
first hour (in order)
  1. ISOLATE affected hosts (network off, power ON — preserve RAM/keys)
  2. isolate shared storage; disable the spreading account/mechanism
  3. capture memory where feasible (keys? attribution?)
  4. scope: hosts / shares / accounts + identify the family
  5. check for DATA THEFT (double extortion -> it's a breach, not just downtime)
  6. locate + verify OFFLINE/IMMUTABLE backups (are they clean? test-restore)

pay-or-not = executive/legal/insurer decision, NOT technical. factors:
  - no guarantee decryption works even if paid
  - funds crime; possible sanctions/legal exposure
  - you still rebuild every compromised system regardless
  - check for a free decryptor (family-specific) first

recovery
  rebuild from known-good, reset creds broadly, close entry vector,
  restore in controlled order. never trust a decrypted-in-place host.
```

## Reading the situation

- **Encryption still in progress** = containment speed is everything; isolate aggressively now, memory capture second. Losses scale with delay.
- **Signs of data exfiltration** = reclassify as a breach with notification duties, not merely an availability event — this changes the legal and comms path entirely.
- **Backups that are offline/immutable and test-restore cleanly** = you have leverage to refuse the ransom and a real recovery path. This is the single biggest factor in the outcome.
- **Backups also encrypted or nonexistent** = the hard case; recovery options narrow, and the pay decision gets harder — escalate immediately with that reality stated plainly.
- **A known family with a public decryptor** = you may recover without paying or full restore; check before any drastic action.

## Pitfalls

- **Powering off infected hosts.** Loses memory that may hold keys and attribution, and doesn't help recovery. Network-isolate, keep powered on.
- **Restoring before confirming backups are clean.** Restoring from a backup the attacker already encrypted or backdoored just re-infects you. Verify and test-restore first.
- **Treating it as availability-only.** Missing the data-theft angle means missing breach-notification obligations — a legal problem on top of the technical one.
- **Making pay/don't-pay a technical call.** It's an executive/legal/insurer decision with major consequences; surface factors and escalate, don't decide it at the keyboard.
- **Decrypting in place and declaring victory.** The attacker was inside; without eradication and credential resets, they come back.

## References

- NIST SP 800-61r2 (incident handling) and NIST ransomware guidance
- CISA #StopRansomware Guide
- No More Ransom project (family identification and free decryptors)
- FBI/CISA guidance on ransom payment considerations
