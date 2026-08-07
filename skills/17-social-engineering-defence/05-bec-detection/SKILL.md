---
name: bec-detection
domain: 17-social-engineering-defence
description: Use when defending against business email compromise — the high-value fraud where an attacker impersonates an executive or vendor to redirect payments, and why it evades normal filters.
difficulty: intermediate
tags: [social-engineering, bec, fraud, email, payments]
tools: []
---

## Purpose

Business Email Compromise is among the costliest cybercrimes — an attacker impersonates a trusted party (an executive, a vendor, a lawyer) to trick someone into wiring money or changing payment details. What makes BEC dangerous is that it often carries no malware and no malicious link, so it sails past technical filters; it's pure social engineering exploiting trust and process. This skill covers detecting and preventing BEC, which requires process controls as much as technical ones.

## When to use it

Defending against payment fraud and executive/vendor impersonation, especially in organisations that handle wire transfers, vendor payments, or payroll changes. BEC's high financial impact and its evasion of normal email security make it a priority distinct from malware-based phishing.

## Why BEC evades normal defences

- **No payload** — BEC emails often have no malware, no attachment, no malicious link; they're plain text asking for a wire transfer or a payment-detail change. So attachment/link scanning and malware detonation catch nothing.
- **Exploits trust and urgency** — the email impersonates authority (the CEO) or a trusted relationship (a vendor) and adds urgency ("wire this today, I'm in a meeting"), pressuring the target to bypass normal process.
- **Often uses legitimate-looking channels** — a lookalike domain, a compromised real account (from account takeover), or a reply-to redirect. It can even come from a genuinely compromised mailbox, passing all email authentication.

## Procedure

1. **Recognise BEC needs process controls, not just technical ones — the key insight.** Because BEC carries no payload, technical email security alone can't stop it; the primary defence is *process*. The single most effective control: **out-of-band verification** for payment changes and unusual transfers — confirm any request to move money or change payment details via a separate, known channel (a phone call to a known number), never by replying to the email.
2. **Apply technical controls where they help** — email authentication (SPF/DKIM/DMARC) stops domain spoofing (a partial defence — it doesn't stop lookalike domains or compromised accounts), and detection can flag lookalike sender domains, new-domain senders, reply-to mismatches, and unusual language/urgency. These reduce but don't eliminate BEC.
3. **Detect the BEC signals.** Flag: display-name impersonation of executives, lookalike/newly-registered sender domains, `Reply-To` differing from `From` (replies redirected to the attacker), requests to change payment details or wire money, urgency + secrecy ("don't tell anyone, handle this directly"), and requests that bypass normal process.
4. **Watch for account-takeover BEC — the hardest case.** When an attacker compromises a *real* internal or vendor mailbox (via credential phishing), the BEC email comes from the genuine account and passes all authentication. Detection here relies on behavioural anomalies (unusual login, mailbox rules the attacker added — the account-compromise IR skill) and, again, out-of-band verification of the request.
5. **Build process controls for money movement** — dual authorisation for payments, out-of-band confirmation of payment-detail changes, and callback verification to known numbers (not numbers in the email). These process controls are what actually stop BEC when the email itself looks legitimate.
6. **Train finance/payment staff specifically** — they're the targets; awareness that a CEO/vendor payment request needs out-of-band verification regardless of how legitimate it looks is the human defence.
7. **Respond to a suspected BEC** — if a transfer was made, act fast (contact the bank to attempt recall — speed matters), and investigate for account compromise (was a mailbox taken over?).

## Cheatsheet

```
BEC = impersonate a trusted party (exec/vendor/lawyer) -> redirect payments. among costliest cybercrimes.
  DANGER: often NO malware / NO link -> sails past technical filters. pure social engineering (trust + process).

why it evades: no payload (scanning catches nothing) | exploits trust + URGENCY
  | legitimate-looking channel (lookalike domain / COMPROMISED real account -> passes auth)

KEY INSIGHT: needs PROCESS controls, not just technical
  #1 control: OUT-OF-BAND VERIFICATION of payment changes / unusual transfers
    (call a KNOWN number — NOT reply to the email, NOT a number in the email)

technical (partial): SPF/DKIM/DMARC (stops domain spoof, not lookalikes/compromised accounts)
  + flag: display-name exec impersonation | lookalike/new sender domain | Reply-To != From
    | payment-change/wire request | urgency + SECRECY | bypasses normal process

hardest case: ACCOUNT-TAKEOVER BEC (real compromised mailbox -> passes all auth)
  -> behavioural anomalies (odd login, added mailbox rules) + out-of-band verification

process controls: DUAL AUTHORISATION | out-of-band confirmation | callback to KNOWN numbers
train FINANCE/payment staff (the targets)
respond: transfer made? -> bank recall FAST + investigate account compromise
```

## Reading the risk

- **A payment/wire request with urgency and secrecy from an "executive"** = a classic BEC pattern; the urgency and "handle this directly, don't tell anyone" are designed to bypass normal process. Out-of-band verification is the defence, regardless of how legitimate it looks.
- **A request to change vendor payment details** = a top BEC vector (redirecting legitimate payments to the attacker); must be confirmed out-of-band via a known channel, never by replying. Payment-detail changes are the highest-risk requests.
- **BEC from a genuinely compromised mailbox** = the hardest case — it passes all email authentication because it *is* the real account. Technical controls fail here; behavioural anomaly detection and out-of-band verification are what catch it.
- **Reliance on email authentication alone** = insufficient for BEC; SPF/DKIM/DMARC stop domain spoofing but not lookalike domains or compromised accounts. BEC needs process controls.
- **`Reply-To` differing from `From`, or a lookalike sender domain** = detectable BEC signals; flag them, though they don't cover the compromised-account case.
- **Out-of-band verification and dual authorisation for money movement** = the controls that actually stop BEC when the email looks legitimate; the process defence is the real protection.

## Pitfalls

- **Relying on technical email security alone.** BEC carries no payload, so scanning and detonation catch nothing; the primary defence is process (out-of-band verification), not technical controls. This is the core mistake.
- **Verifying by replying to the email or calling a number in it.** The attacker controls both; verification must be via a *separately-known* channel (a known phone number). Out-of-band means genuinely out of band.
- **Assuming email authentication stops BEC.** SPF/DKIM/DMARC stop domain spoofing but not lookalike domains or compromised accounts; a BEC from a real hijacked mailbox passes all of it.
- **Not training finance/payment staff.** They're the targets; without awareness that payment requests need out-of-band verification regardless of apparent legitimacy, the human defence is absent.
- **Slow response to a completed transfer.** Recall attempts are time-sensitive; contact the bank fast, and investigate whether a mailbox was compromised.

## References

- FBI/IC3 Business Email Compromise reports and guidance
- The spf-dkim-dmarc, phishing-email-analysis, and IR account-compromise-response skills
- CISA BEC prevention guidance (out-of-band verification, dual authorisation)
- MITRE ATT&CK — social engineering / phishing techniques
