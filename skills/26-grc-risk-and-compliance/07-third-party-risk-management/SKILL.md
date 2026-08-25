---
format: "v2"
name: "third-party-risk-management"
title: "Third Party Risk Management"
title_fr: "Gestion du risque tiers"
description: "Use when running a vendor/third-party risk programme — assessing and monitoring the security of the vendors and suppliers whose access and services become your risk."
description_fr: "À utiliser pour faire vivre un programme de risque fournisseurs/tiers — évaluer et surveiller la sécurité des fournisseurs et prestataires dont l'accès et les services deviennent votre risque."
domain: "26-grc-risk-and-compliance"
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

Your security is only as strong as the vendors you depend on — a supplier with access to your systems or data, or whose service you rely on, becomes part of your attack surface, and their breach can become yours. Third-party risk management (TPRM) is the programme for assessing and monitoring vendor security across the relationship. This skill covers running TPRM from the governance/programme angle (the software-supply-chain domain covers the technical dependency side; this is the vendor/organisational side).

### When to use it

When the organisation depends on vendors with access to its data, systems, or critical services — which is nearly all organisations. It's increasingly a *regulatory* requirement (NIS2, DORA) as well as good practice, and vendor breaches are a common intrusion path (the vendor-compromise supply-chain angle).

### Procedure

1. **Inventory your third parties and their risk.** Know which vendors you use, and — critically — which have access to your data/systems or provide critical services. Not all vendors are equal: a vendor with admin access to your systems or holding your customer data is high-risk; a vendor providing office snacks isn't. Prioritise by access and criticality.
2. **Assess vendors proportionate to their risk — the key efficiency.** Don't assess every vendor identically; scale the depth of assessment to the risk. A high-risk vendor (deep access, critical service, sensitive data) warrants a thorough security review; a low-risk one needs little. Uniform heavy assessment of all vendors wastes effort and delays the business; proportionate assessment focuses it where it matters.
3. **Assess vendor security before onboarding.** For significant vendors, evaluate their security posture *before* depending on them — questionnaires (SIG, CAIQ), their certifications/attestations (SOC 2, ISO 27001), and evidence of their controls. Assessing after you're already dependent is too late to influence the decision.
4. **Use their attestations to reduce effort.** A vendor's SOC 2 report or ISO 27001 certification is evidence of their security you can rely on rather than assessing from scratch; leverage these to make assessment efficient (and require them from high-risk vendors).
5. **Address the access and data they have.** Beyond assessing their posture, control the risk directly: least-privilege access for vendors, monitoring their access, and contractual security requirements (breach notification, security obligations, right to audit). The access itself is the risk — limit and monitor it.
6. **Monitor continuously, not just at onboarding.** A vendor secure at onboarding can degrade, get breached, or change; the risk is ongoing. Reassess periodically (proportionate to risk), monitor for vendor breaches (news, breach notifications), and use continuous vendor-risk-rating services where warranted. A one-time assessment misses the risk that emerges over the relationship.
7. **Plan for vendor incidents.** A vendor breach can be your incident (their compromise reaches your data/systems); include vendor-breach scenarios in incident response, and ensure contracts require timely breach notification so you learn quickly.

### Cheatsheet

```
your security = as strong as your VENDORS. a vendor with access/data = your attack surface ; their breach = yours.
  TPRM = assess + monitor vendor security across the relationship (regulatory now: NIS2/DORA)

do
  1. INVENTORY third parties + which have ACCESS to data/systems / critical service
       (not all equal — admin-access/data-holding vendor = high ; snacks = not) -> prioritise
  2. ASSESS PROPORTIONATE to risk (key efficiency): high-risk = thorough ; low = light
       (uniform heavy assessment wastes effort + delays business)
  3. assess BEFORE onboarding (questionnaires SIG/CAIQ, certs SOC2/ISO27001, control evidence)
       (assessing after you depend on them = too late)
  4. USE their ATTESTATIONS (SOC2/ISO27001 = evidence, rely on it not re-assess) — require from high-risk
  5. address the ACCESS/DATA directly: least-privilege vendor access + monitor + CONTRACTUAL security
       (breach notification, obligations, right to audit) — the access IS the risk
  6. MONITOR CONTINUOUSLY (secure at onboarding can degrade/get breached) — reassess, watch for breaches
  7. plan for VENDOR INCIDENTS (their breach = your incident ; contracts require timely notification)
```

### Reading the programme

- **Assessing every vendor identically** = wasted effort and business delay; scale assessment depth to the vendor's risk (access, criticality, data). Proportionate assessment focuses effort where it matters — uniform heavy assessment is a common inefficiency.
- **A high-risk vendor** (deep system access, holds sensitive data, critical service) = warrants thorough review before onboarding and ongoing monitoring; their breach directly becomes yours. Prioritise these.
- **Assessing a vendor only after depending on them** = too late to influence the decision; assess significant vendors before onboarding, when you can still walk away or require improvements.
- **Relying on a vendor's SOC 2 / ISO 27001** = efficient use of their attestation as evidence; requiring these from high-risk vendors reduces your assessment burden.
- **A one-time onboarding assessment with no ongoing monitoring** = misses the risk that emerges over the relationship (degradation, breaches, changes); the risk is continuous. Reassess and monitor.
- **Vendor access unrestricted and unmonitored** = the access itself is the risk regardless of posture; least-privilege, monitoring, and contractual security controls address it directly.
- **Prioritised, proportionate, pre-onboarding, continuously-monitored vendor assessment with contractual controls** = TPRM that manages the vendor attack surface.

### Pitfalls

- **Assessing all vendors identically.** Uniform heavy assessment wastes effort and delays the business; scale depth to the vendor's risk. Proportionate assessment is the key efficiency.
- **Assessing after onboarding.** Too late to influence the decision; assess significant vendors before you depend on them.
- **One-time assessment.** Vendors degrade, get breached, and change; the risk is ongoing. Monitor continuously and reassess proportionate to risk.
- **Ignoring the access itself.** Even a well-assessed vendor's access is risk; apply least-privilege, monitoring, and contractual security controls directly.
- **Not planning for vendor breaches.** A vendor's compromise can be your incident; include it in IR and require timely breach notification contractually.
- **Not leveraging attestations.** Re-assessing a vendor who has a SOC 2 / ISO 27001 wastes effort; use their attestations as evidence.

### References

- The software-supply-chain third-party-risk skill (technical dependency side) and the NIS2/DORA obligations
- Standardised assessments: SIG (Shared Assessments), CAIQ (Cloud Security Alliance)
- The soc-2-readiness and iso-27001 skills (vendor attestations you rely on)
- NIST SP 800-161 (supply-chain risk management)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.