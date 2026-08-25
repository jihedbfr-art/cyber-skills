---
format: "v2"
name: "spf-dkim-dmarc"
title: "Spf Dkim Dmarc"
title_fr: "SPF, DKIM, DMARC"
description: "Use when configuring email authentication to stop spoofing of your domain — SPF, DKIM, and DMARC, the records that keep attackers from sending mail as you."
description_fr: "À utiliser pour configurer l'authentification des e-mails et empêcher l'usurpation de votre domaine — SPF, DKIM et DMARC, les enregistrements qui empêchent les attaquants d'envoyer du courrier en se faisant passer pour vous."
domain: "17-social-engineering-defence"
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

The easiest phishing is the one that comes *from your own domain* — an email that appears to be from your CEO or your IT department, because email by default lets anyone claim any From address. SPF, DKIM, and DMARC are the three records that let receiving servers verify a message genuinely came from your domain and reject the ones that don't. This skill covers configuring them to stop attackers spoofing your domain, closing a major phishing vector.

### When to use it

Hardening your organisation's email against domain spoofing, and as a defensive follow-up whenever phishing analysis (that skill) shows your domain being spoofed. It's a high-value, foundational anti-phishing control that many organisations still have misconfigured.

### The three mechanisms (how they work together)

- **SPF (Sender Policy Framework)** — a DNS record listing which servers are allowed to send mail for your domain. Receivers check whether the sending server is on the list.
- **DKIM (DomainKeys Identified Mail)** — a cryptographic signature added to outgoing mail; receivers verify it against a public key in your DNS, proving the message wasn't forged or altered.
- **DMARC** — ties SPF and DKIM to the visible From address (alignment), tells receivers what to do with mail that fails (none/quarantine/reject), and provides reporting. DMARC is what actually enforces the anti-spoofing — SPF and DKIM alone don't tell receivers to reject failures.

### Procedure

1. **Publish an SPF record** listing all legitimate sending sources (your mail servers, and any third parties that send as you — marketing platforms, ticketing systems). An incomplete SPF breaks legitimate mail; a too-permissive one (`+all`) authorises anyone. End with `-all` (hard fail) once you're confident the list is complete:
   ```
   dig TXT example.com | grep spf        # v=spf1 include:... -all
   ```
2. **Set up DKIM signing** on your outgoing mail, publishing the public key in DNS. Every legitimate message gets signed; forged ones can't be.
3. **Publish a DMARC record with alignment and a policy** — this is what enforces anti-spoofing. Start at `p=none` (monitor only, collect reports) to see what's sending as you without breaking anything, then move to `p=quarantine` and finally `p=reject` once legitimate mail is passing:
   ```
   dig TXT _dmarc.example.com            # v=DMARC1; p=reject; rua=mailto:...
   ```
4. **Use DMARC reports to find all your legitimate senders.** The `rua` reports show every source sending as your domain — including forgotten third parties and the spoofing attempts. Use them to complete SPF/DKIM before enforcing, and to see spoofing of your domain.
5. **Progress to enforcement (`p=reject`) — the goal.** `p=none` monitors but doesn't stop spoofing; only `quarantine`/`reject` actually block spoofed mail. Many organisations stall at `p=none` and remain spoofable. Getting to `reject` is what closes the vector.
6. **Cover all your domains**, including parked/unused ones — attackers spoof any domain you own, and unused domains that can't send mail should have a restrictive policy so they can't be abused.

### Cheatsheet

```
email lets ANYONE claim any From -> easiest phishing = FROM YOUR OWN DOMAIN
  SPF + DKIM + DMARC = receivers verify mail really came from you, reject what doesn't

SPF    DNS list of servers allowed to send for your domain ; end with -all (hard fail)
  (incomplete = breaks legit mail ; +all = authorises anyone)
DKIM   crypto signature on outgoing mail, verified vs public key in DNS (unforgeable)
DMARC  ties SPF/DKIM to visible From (alignment) + policy (none/quarantine/REJECT) + reporting
  -> DMARC is what ENFORCES anti-spoofing (SPF/DKIM alone don't tell receivers to reject)

rollout
  1. SPF: list ALL legit senders (+ third parties) -> -all when complete
  2. DKIM signing + public key in DNS
  3. DMARC p=none (monitor, collect rua reports) -> find all senders + see spoofing
  4. complete SPF/DKIM from reports -> p=quarantine -> p=REJECT (the goal)
     (stalling at p=none = still spoofable — the common failure)
  5. cover ALL domains incl. PARKED (restrictive policy so they can't be abused)
```

### Reading the configuration

- **DMARC at `p=none` (or no DMARC)** = your domain is spoofable; monitoring doesn't stop spoofing, only `quarantine`/`reject` does. Many organisations stall here and remain vulnerable — getting to `reject` is the goal and the most common gap.
- **No DMARC at all** = SPF/DKIM without DMARC don't enforce alignment or tell receivers to reject failures; the domain can still be spoofed. DMARC is the enforcing layer.
- **SPF `+all` or overly permissive** = authorises anyone to send as you; effectively no SPF. Use `-all` with a complete sender list.
- **Incomplete SPF/DKIM** breaking legitimate mail = why you monitor with `p=none` and DMARC reports first — to find all legitimate senders before enforcing. Rushing to `reject` breaks mail.
- **Parked/unused domains without a restrictive policy** = spoofable domains you own but forgot; attackers use them. Set a reject policy on domains that shouldn't send mail.
- **`p=reject` with complete SPF/DKIM across all domains** = the goal state; your domains can't be spoofed, closing a major phishing vector.

### Pitfalls

- **Stalling at `p=none`.** Monitoring mode doesn't stop spoofing; the domain remains spoofable. The whole point is reaching `quarantine`/`reject`. This is the most common failure.
- **No DMARC, only SPF/DKIM.** Without DMARC, there's no alignment enforcement and no instruction to reject failures; the domain is still spoofable. DMARC is what enforces.
- **Rushing to `reject` with incomplete SPF/DKIM.** It breaks legitimate mail (forgotten third-party senders fail). Monitor with reports first, complete the sender list, then enforce.
- **Overly permissive SPF (`+all`).** Authorises anyone; use `-all` with a complete list.
- **Forgetting parked domains.** Attackers spoof any domain you own; unused domains need a restrictive policy so they can't be abused.

### References

- SPF (RFC 7208), DKIM (RFC 6376), DMARC (RFC 7489) specifications
- The phishing-email-analysis skill (spoofing detection) and M3AAWG email authentication best practices
- DMARC deployment guides and reporting analysers
- The bec-detection skill (email authentication is a partial BEC defence)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.