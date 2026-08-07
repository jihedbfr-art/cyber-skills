---
name: url-and-attachment-detonation
domain: 17-social-engineering-defence
description: Use when safely analysing suspicious links and attachments — detonating them in isolation to determine if they're malicious without exposing yourself or the network.
difficulty: intermediate
tags: [social-engineering, phishing, detonation, sandbox, analysis]
tools: [any.run, cuckoo, urlscan]
---

## Purpose

When a suspicious email is reported, someone has to determine whether the link or attachment is actually malicious — and doing that safely, without infecting yourself or triggering the payload on a real system, is the skill. Detonation means analysing a URL or file in an isolated environment where it can do its worst harmlessly. This skill covers safely detonating and analysing phishing links and attachments, the analysis step behind phishing triage and response.

## When to use it

Analysing reported phishing (the phishing-email-analysis skill hands off here), or any suspicious link/file. The overriding principle is safety: these are potentially malicious, so they're analysed in isolation, never opened on a normal workstation.

## Procedure

1. **Never open it on your workstation — the safety rule.** A suspicious link or attachment is potentially malicious; opening it on a normal system infects it or triggers the payload. All analysis happens in isolation — a sandbox or an analysis service that detonates it away from your environment.
2. **Analyse URLs without visiting them directly.** Submit the URL to a service that fetches and analyses it in isolation (urlscan.io, VirusTotal, a sandbox), showing the destination, any redirect chain, and whether it's a credential-harvest page or malware host — without your browser touching it:
   ```
   # submit URL to urlscan.io / VirusTotal / a sandbox — read the verdict + screenshot
   ```
   Watch for lookalike domains, credential-harvesting login pages, and redirect chains that end at malware.
3. **Detonate attachments in a sandbox.** Submit the file to an automated malware sandbox (any.run, Joe Sandbox, or a self-hosted Cuckoo/CAPE — the malware domain's dynamic-analysis skill) that runs it and records its behaviour: what it drops, what it connects to, what it does. This reveals whether it's malicious and what it would do.
4. **Check reputation first (by hash/URL).** Before deep detonation, a reputation lookup (VirusTotal) may already identify a known-malicious file/URL, saving time. But absence of detections isn't proof of safety for a targeted attack — detonate if unsure.
5. **Extract indicators.** From the detonation, capture the IoCs — the malicious URL, the payload hash, the C2 destination, dropped filenames (the malware extracting-iocs skill) — to block them and hunt for others who received the same phish.
6. **Mind confidentiality and evasion.** Public sandboxes/services expose the sample — don't submit confidential documents or targeted samples that could tip off an attacker to public services; use a self-hosted sandbox for sensitive analysis. Also, some malware detects sandboxes and plays dead (the malware evasion angle).
7. **Feed the verdict into response.** A confirmed-malicious link/attachment drives blocking (URL/hash/sender), pulling the email from other mailboxes, and hunting for other recipients — the phishing-analysis and IR response.

## Cheatsheet

```
someone must decide: is this link/attachment malicious? — SAFELY (no self-infection / payload trigger)
  detonation = analyse in ISOLATION where it can do its worst harmlessly

SAFETY RULE: NEVER open on your workstation. isolation only (sandbox / analysis service).

URLs (don't visit directly)
  submit to urlscan.io / VirusTotal / sandbox -> destination, redirect chain, cred-harvest page?
  watch: lookalike domains, credential-harvest login, redirect -> malware

ATTACHMENTS (sandbox detonate)
  any.run / Joe Sandbox / self-hosted Cuckoo/CAPE -> drops? connects? behaviour?
  (malware dynamic-analysis skill)

reputation FIRST (hash/URL on VT) — may already be known-bad (saves time)
  but no detections != safe for a TARGETED attack -> detonate if unsure

extract IoCs (URL, hash, C2, dropped files) -> block + hunt other recipients
CONFIDENTIALITY: public services EXPOSE the sample -> self-host for sensitive/targeted samples
  (+ some malware detects sandboxes -> plays dead)
verdict -> response (block, pull email, hunt recipients)
```

## Reading the analysis

- **A URL resolving to a credential-harvest login page on a lookalike domain** = confirmed phishing; the fake login is designed to steal credentials. Capture the URL for blocking and note the harvesting intent. A common, high-confidence verdict.
- **An attachment that drops files and connects to a C2 in the sandbox** = confirmed malicious; the detonation reveals the payload's behaviour and its indicators. Extract the IoCs.
- **A known-malicious hash/URL from reputation lookup** = you may have your answer without full detonation; but for a targeted attack, absence of detections isn't safety — detonate if unsure.
- **A sample doing nothing in the sandbox** = possibly benign, but suspect sandbox evasion (the malware plays dead when it detects analysis); don't conclude harmless from inactivity alone.
- **A redirect chain ending at malware** = the URL looked benign but redirects to a payload; the isolated fetch reveals the real destination your browser would have reached.
- **Confirmed-malicious verdict with extracted IoCs** = drives the response (block URL/hash/sender, pull the email, hunt other recipients) — the point of the analysis.

## Pitfalls

- **Opening it on your workstation.** The cardinal safety error — a suspicious link/attachment is potentially malicious; opening it infects you or triggers the payload. Isolation only.
- **Visiting URLs directly to "check".** Your browser touching the URL can harvest, exploit, or confirm your address; submit it to an isolation service instead.
- **Submitting confidential/targeted samples to public services.** They expose the sample and can tip off a targeted attacker; use a self-hosted sandbox for sensitive analysis.
- **Concluding "safe" from no sandbox activity.** Evasive malware plays dead when it detects a sandbox; inactivity isn't proof of safety.
- **Not extracting IoCs.** A verdict without indicators doesn't drive blocking and hunting; capture the URL, hash, and C2 for response.
- **Trusting reputation absence as safety.** For targeted attacks, no detections doesn't mean benign; detonate if unsure.

## References

- urlscan.io, VirusTotal, any.run, Joe Sandbox, and Cuckoo/CAPE documentation
- The phishing-email-analysis, malware dynamic-analysis-sandboxing, and extracting-iocs skills
- The building-a-malware-lab skill (self-hosted sandbox for sensitive samples)
- SANS phishing analysis resources
