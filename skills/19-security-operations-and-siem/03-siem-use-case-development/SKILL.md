---
format: "v2"
name: "siem-use-case-development"
title: "Siem Use Case Development"
title_fr: "Développement de cas d'usage SIEM"
description: "Use when developing SIEM use cases — the structured process of turning a security requirement into a deployed, documented detection with a response, not just a raw rule."
description_fr: "À utiliser pour développer des cas d'usage SIEM — le processus structuré qui transforme un besoin de sécurité en une détection déployée, documentée et associée à une réponse, pas juste une règle brute."
domain: "19-security-operations-and-siem"
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

A SIEM "use case" is more than a detection rule — it's the whole package: the threat it addresses, the logic that detects it, the log sources it needs, the response when it fires, and the ownership to maintain it. Developing use cases well is what turns a SIEM from a rule graveyard into an operational capability. This skill covers the structured process of building use cases that are actually usable by the SOC, bridging detection engineering and SOC operations.

### When to use it

Building out a SIEM's detection capability in an organised way, rather than accumulating ad-hoc rules. It sits between detection engineering (the rule logic) and SOC operations (working the alerts) — a use case is a detection wrapped in everything the SOC needs to act on it.

### What a use case includes

A complete use case answers more than "what does the rule match":

- **The requirement / threat** — what risk or technique it addresses (mapped to ATT&CK), and why it matters to this org.
- **The detection logic** — the rule itself (built and tested per the detection skills).
- **The log sources** — what telemetry it depends on (and confirmation it's collected).
- **The response** — what the analyst does when it fires: triage steps, escalation criteria, remediation (a runbook, not just an alert).
- **The metadata** — owner, severity, false-positive expectations, tuning notes, and status (the detection-as-code discipline).

### Procedure

1. **Start from a requirement, not a rule.** A use case begins with "we need to detect X because Y" — a threat from the threat model, a compliance need, a lesson from an incident. Requirement-driven use cases address real needs; rule-first ones detect whatever was easy.
2. **Confirm the data exists** before building — the required log sources must be collected and parsed (log-source-coverage and pipeline skills). A use case for telemetry you don't have is a non-starter.
3. **Build and test the detection** (the detection-engineering skills) — write it, tune it for false positives, and validate it fires on the real technique. The logic is one component, not the whole use case.
4. **Write the response runbook — the part that makes it operational.** Define what the analyst does when it fires: how to triage, what confirms true vs false positive, when to escalate, and how to remediate. A detection without a response leaves the SOC to improvise; the runbook is what turns an alert into action (ties into the SOC triage skill).
5. **Assign severity and ownership.** Rate the alert's priority realistically (not everything critical), and give the use case an owner responsible for maintaining and tuning it.
6. **Deploy through the lifecycle and document.** Move it experimental → production as it proves out, and document all of the above so it's maintainable and the SOC knows how to work it.
7. **Review and retire.** Use cases age — the threat changes, the environment shifts, false positives creep. Review them periodically and retire ones that no longer earn their keep, so the SIEM doesn't become a rule graveyard.

### Cheatsheet

```
a use case = detection + everything the SOC needs to act on it (not just a rule)

components
  REQUIREMENT/threat   what risk/technique (ATT&CK) + why it matters here
  DETECTION LOGIC      the rule (built + tuned + tested — detection skills)
  LOG SOURCES          telemetry it needs (confirmed collected)
  RESPONSE runbook     triage steps, TP vs FP, escalation, remediation  <- makes it operational
  METADATA             owner, severity, FP expectations, tuning notes, status

process
  1. start from a REQUIREMENT (not a rule) — real need, not what's easy
  2. confirm data EXISTS (log-source-coverage) before building
  3. build + test the detection (detection engineering)
  4. write the RESPONSE runbook (detection without response = SOC improvises)
  5. severity + OWNER (realistic priority; someone maintains it)
  6. deploy through lifecycle + document
  7. REVIEW + retire aging use cases (else SIEM = rule graveyard)
```

### Reading a use case

- **A deployed rule with no response runbook** = incomplete; when it fires, the SOC improvises, triage is slow and inconsistent. The response is what makes a detection a use case. Add the runbook.
- **A rule-first use case** (built because it was easy, not because of a requirement) = may not address a real need; requirement-driven development ensures the SIEM detects what matters.
- **A use case for uncollected telemetry** = it can never fire; confirm the data exists first (a common wasted-effort mistake).
- **No owner** = it won't be maintained; false positives creep, the threat evolves, and it rots. Ownership keeps it alive.
- **A SIEM full of old, untuned, unowned rules** = a rule graveyard; without periodic review and retirement, use cases accumulate into noise. Review and prune.
- **Requirement-driven, data-confirmed, tested, runbook-backed, owned use cases** = an operational SIEM the SOC can actually work.

### Pitfalls

- **Building rules, not use cases.** A rule without response guidance, ownership, and documentation leaves the SOC to figure out what to do when it fires. Wrap the detection in the operational package.
- **Rule-first instead of requirement-first.** Detecting whatever's easy rather than what the threat model needs produces a SIEM aimed at nothing in particular.
- **Skipping the data check.** A use case for telemetry you don't collect never works; confirm the sources first.
- **No response runbook.** The most common gap — an accurate detection with no defined response is only half a capability; triage suffers.
- **Never retiring use cases.** They age into a rule graveyard of noisy, irrelevant, unowned rules. Review and prune periodically.

### References

- The detection-engineering domain (rule logic, testing, mapping) and log-source-coverage skill
- The SOC alert-triage-workflow skill (the response side)
- MITRE ATT&CK (requirement/threat mapping)
- SANS SIEM use-case development resources

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.