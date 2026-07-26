---
name: account-compromise-response
domain: 22-incident-response
description: Use when a user or service account is suspected compromised — containing the stolen identity, understanding what it touched, and restoring it without leaving the attacker a way back.
difficulty: intermediate
tags: [incident-response, account-compromise, identity, credentials]
tools: []
---

## Purpose

A compromised account is the most common incident type — a phished user, a leaked key, a stolen session. The attacker isn't exploiting anything; they're logged in as a legitimate identity, which makes their actions blend in. This skill covers responding to account compromise: cutting off the access, working out what the identity did, and closing the door — including the persistence attackers add so a password reset alone doesn't evict them.

## When to use it

When triage points to a compromised identity: impossible-travel logins, activity the real user disowns, a leaked credential found, mailbox rules nobody set. It applies to user accounts, admin accounts, and service accounts/API keys alike.

## Procedure

1. **Contain the identity fast.** Disable the account or force a session revocation — and note that a password reset **alone does not** kill active sessions or tokens. You must invalidate existing sessions, refresh tokens, and API keys too, or the attacker stays logged in through the reset.
2. **Reset credentials and rotate secrets** the identity holds — password, API keys, app passwords, and any secrets the account could access.
3. **Hunt for attacker persistence added under the identity** — this is the step that's routinely missed:
   - **mailbox rules** (auto-forward to an external address, delete-on-arrival to hide replies) — a hallmark of business email compromise.
   - **OAuth app grants / connected apps** the attacker authorised to retain access even after a password reset.
   - **added MFA devices / recovery methods** the attacker registered so they can get back in.
   - **new API keys, app passwords, delegate access**, or added account recovery emails/phones.
4. **Scope what the identity did.** Review the account's activity during the compromise window — data accessed, emails sent, changes made, systems reached. A compromised account is a pivot; check for lateral movement to other accounts or systems.
5. **Check blast radius by privilege.** A compromised admin or service account is far worse than a standard user — it may have created other footholds (see credential-dumping/eradication for domain-wide cases).
6. **Restore and re-enable** the account with fresh credentials and MFA, and **monitor** it closely afterward for signs the attacker returns.
7. **Notify** as required — the user, and anyone affected by what the account did (especially if it sent phishing or accessed others' data).

## Cheatsheet

```
contain (fast)
  [ ] disable account / force sign-out EVERYWHERE
  [ ] revoke sessions + refresh tokens + API keys  (reset alone != logout)
  [ ] reset password, rotate the account's secrets

hunt persistence added by the attacker (the missed step)
  [ ] mailbox auto-forward / hide-reply rules   (BEC hallmark)
  [ ] rogue OAuth app / connected-app grants
  [ ] attacker-added MFA devices / recovery methods
  [ ] new API keys, app passwords, delegate/recovery emails

scope
  [ ] activity in the compromise window: data, emails sent, changes, systems reached
  [ ] lateral movement to other accounts/systems?
  [ ] privilege: admin/service account = wider blast radius

restore: fresh creds + MFA, then heightened monitoring for return
```

## Reading the situation

- **A password reset done without revoking sessions/tokens** = the attacker likely still has access; the reset gave false closure. Revoke everything.
- **A mailbox auto-forward rule** = business email compromise; the attacker is siphoning mail and may have hidden their tracks. High-signal, commonly overlooked.
- **A rogue OAuth grant or attacker-added MFA device** = persistence that survives a password change — the account is still owned until these are removed.
- **A compromised admin or service account** = treat as a potential wider intrusion; it can create footholds a user account can't (escalate to eradication).
- **Emails sent or data accessed during the window** = downstream impact (further phishing, a data breach) that needs its own response and notification.

## Pitfalls

- **Password reset as the whole response.** It doesn't revoke live sessions, tokens, or attacker-added persistence. The account can still be owned after it.
- **Missing mailbox rules and OAuth grants.** These are how attackers stay in after a reset; skipping the persistence hunt means the "fixed" account is still compromised.
- **Not scoping activity.** A compromised account is a pivot; ignoring what it did misses lateral movement and downstream harm.
- **Under-rating service/admin accounts.** Their compromise is often a wider incident, not a single-account cleanup.

## References

- NIST SP 800-61r2 (incident handling)
- CISA and Microsoft guidance on business email compromise / account compromise
- MITRE ATT&CK — Valid Accounts (T1078), Account Manipulation (T1098)
