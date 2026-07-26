---
name: phishing-email-analysis
domain: 17-social-engineering-defence
description: Use when a suspicious email needs analysing — reading headers, links, and attachments to decide if it's phishing and what to do about it, safely.
difficulty: beginner
tags: [social-engineering, phishing, email, headers, analysis]
tools: [thunderbird, urlscan, virustotal]
---

## Purpose

A reported email lands on your desk: is it phishing, and if so, how bad? This skill covers dissecting a suspicious message — headers, sender authenticity, links, and attachments — without setting off whatever it's carrying. It's the practical triage a SOC or IT team runs dozens of times a week.

## When to use it

Any time a user reports a suspicious email, or one gets flagged by a filter and needs a human verdict. Also the analysis step behind takedowns and awareness feedback.

Work on the email as **data**: examine headers and URLs in tools, never click links or open attachments on your normal machine. Detonate anything live only in an isolated sandbox.

## Procedure

1. Get the **original message with full headers** (the raw `.eml`/`.msg`, not a forwarded screenshot — forwarding strips the evidence). In most clients: "show original" / "view source".
2. Read the **authentication results** in the headers. `Authentication-Results` shows SPF, DKIM, and DMARC verdicts. A `fail`/`softfail` on a domain that should pass is a strong spoofing signal:
   ```
   Authentication-Results: spf=fail dkim=fail dmarc=fail header.from=bank.com
   ```
3. Trace the **Received chain** from bottom (origin) to top. Does the originating server match the claimed sender's infrastructure, or is it some unrelated host? Mismatch is suspicious.
4. Check the **From vs Return-Path vs Reply-To.** Phishing often has a spoofed display name, a mismatched `Reply-To` (so replies go to the attacker), and a `From` domain that's a lookalike (`bank-secure.com`, `bank.com.evil.net`).
5. Analyse **links without clicking.** Hover/extract the real URL, expand shorteners, and submit it to a URL analysis service that fetches it for you in isolation:
   ```
   # inspect the URL, don't visit it
   urlscan.io / any.run / virustotal — submit the URL, read the verdict
   ```
   Watch for lookalike domains, `@` tricks (`https://real.com@evil.com`), and credential-harvest landing pages.
6. Analyse **attachments** by hash, not by opening. Compute the hash and check reputation; detonate in a sandbox only if you need behaviour:
   ```
   sha256sum attachment.docx    # then look up the hash on VirusTotal
   ```
7. Reach a verdict and act: confirmed phishing → pull it from other mailboxes, block the sender/URL/hash, report for takedown, and feed indicators to detection. Benign → release and note why.

## Cheatsheet

```
header signals
  Authentication-Results: spf/dkim/dmarc = fail  -> likely spoofed
  From vs Return-Path vs Reply-To mismatch        -> replies redirected
  Received chain origin unrelated to sender       -> not who it claims
  display name "IT Support" but domain random      -> impersonation

link / attachment (never open directly)
  hover to reveal real URL; expand shorteners
  lookalike domain / homoglyphs / user@evil.com trick
  submit URL to urlscan/VT; look up file hash on VT
  detonate only in an isolated sandbox

content signals
  urgency + threat ("account will be closed")
  credential / payment request
  unexpected attachment (.htm, .iso, macro-enabled office)
  generic greeting, off-brand tone, subtle grammar errors
```

## Reading the verdict

- **DMARC fail on a domain that publishes DMARC** is close to conclusive spoofing — the sender isn't who the `From` claims.
- **A credential-harvest landing page** (a login form on a lookalike domain) confirms phishing intent; capture the URL for blocking and takedown.
- **A malicious attachment hash** with detections is a confirmed payload — block the hash fleet-wide and hunt for others who received it.
- **Authentication passes but content screams phish** can be a compromised legitimate account (BEC) — pass passes because it really is sent from the real, but hijacked, mailbox. Different response (account compromise), same danger.
- **Genuinely benign** happens — a real but unusual vendor mail. Say so, release it, and note the reasoning so the reporter is encouraged, not punished.

## Turning analysis into defence (the "fix")

- **Block and purge**: remove the message from all mailboxes, block sender/URL/hash at the gateway.
- **Feed indicators** (sender, URLs, hashes) to detection and threat intel so the next wave is caught automatically.
- **Fix your own spoofability**: if attackers spoof *your* domain, deploy SPF/DKIM/DMARC with an enforcing policy (see that skill).
- **Close the loop with the user**: thank them for reporting, and use real examples (sanitised) in awareness training. A strong reporting culture catches what filters miss.

## Pitfalls

- **Analysing a forwarded copy.** Forwarding destroys the original headers you need. Get the raw message.
- **Clicking to "just check".** The link may harvest, exploit, or simply confirm your address is live. Inspect via tools in isolation.
- **Opening attachments on your workstation.** Hash and sandbox instead.
- **Assuming SPF/DKIM pass = safe.** A compromised legitimate account passes authentication. Read the content and context too.
- **Punishing reporters for false alarms.** That trains people to stop reporting — the opposite of what you want.

## References

- OWASP — Phishing / social engineering guidance
- M3AAWG email authentication best practices
- urlscan.io, VirusTotal, and sandbox (any.run) documentation
- APWG phishing reporting resources
