# cyber-skills

A working library of security skills, grouped into 26 domains. Every skill lives in its own folder with a single `SKILL.md` that works two ways: a checklist an engineer can follow, and a procedure an agentic coding assistant can load and run.

[Version française](README.fr.md)

## Why this exists

Most security knowledge on GitHub is either a link dump or a tool list. Useful, but you still have to figure out the *order of operations* yourself — what to check first, what a finding actually means, what to do next.

This repo stores the procedure, not just the pointer. A skill answers three questions: when do I reach for this, what exactly do I run, and how do I read the output.

## The 26 domains

| # | Domain | Focus |
|---|---|---|
| 01 | [OSINT & Reconnaissance](skills/01-osint-and-reconnaissance) | Attack surface discovery from public sources |
| 02 | [Network Security](skills/02-network-security) | Segmentation, scanning, traffic analysis |
| 03 | [Web Application Security](skills/03-web-application-security) | OWASP-class flaws in web apps |
| 04 | [API Security](skills/04-api-security) | REST, GraphQL, gRPC, auth and rate limits |
| 05 | [Mobile Security](skills/05-mobile-security) | Android and iOS app assessment |
| 06 | [Cloud Security](skills/06-cloud-security) | AWS, Azure, GCP posture and misconfiguration |
| 07 | [Container & Kubernetes Security](skills/07-container-and-kubernetes-security) | Images, runtime, cluster hardening |
| 08 | [DevSecOps & CI/CD Security](skills/08-devsecops-and-cicd-security) | Pipeline integrity, secrets, gates |
| 09 | [Software Supply Chain Security](skills/09-software-supply-chain-security) | Dependencies, SBOM, artifact signing |
| 10 | [Secure Code Review](skills/10-secure-code-review) | Reading code for vulnerability patterns |
| 11 | [Identity & Access Management](skills/11-identity-and-access-management) | OAuth2, OIDC, SAML, session handling |
| 12 | [Active Directory & Windows Security](skills/12-active-directory-and-windows-security) | Domain attack paths and hardening |
| 13 | [Linux & Unix Security](skills/13-linux-and-unix-security) | Hardening, privilege boundaries, auditing |
| 14 | [Cryptography & PKI](skills/14-cryptography-and-pki) | Correct use of primitives, certificates, key management |
| 15 | [Vulnerability Management](skills/15-vulnerability-management) | Scanning, triage, prioritisation, SLA |
| 16 | [Red Teaming & Adversary Emulation](skills/16-red-teaming-and-adversary-emulation) | Authorised offensive testing against ATT&CK |
| 17 | [Social Engineering Defence](skills/17-social-engineering-defence) | Phishing analysis, awareness, controls |
| 18 | [Detection Engineering](skills/18-detection-engineering) | Writing and testing detection rules |
| 19 | [Security Operations & SIEM](skills/19-security-operations-and-siem) | Log pipelines, alert triage, on-call |
| 20 | [Threat Hunting](skills/20-threat-hunting) | Hypothesis-driven search across telemetry |
| 21 | [Threat Intelligence](skills/21-threat-intelligence) | Collection, enrichment, actor tracking |
| 22 | [Incident Response](skills/22-incident-response) | Containment, eradication, recovery |
| 23 | [Digital Forensics](skills/23-digital-forensics) | Disk, memory, cloud and mobile artefacts |
| 24 | [Malware Analysis & Reverse Engineering](skills/24-malware-analysis-and-reverse-engineering) | Static and dynamic analysis |
| 25 | [AI & LLM Security](skills/25-ai-and-llm-security) | Prompt injection, model supply chain, agent safety |
| 26 | [GRC, Risk & Compliance](skills/26-grc-risk-and-compliance) | ISO 27001, SOC 2, NIS2, risk registers |

## Using a skill

**As a human.** Open the `SKILL.md`, skip to the cheatsheet, run the commands. The procedure above it explains why the order matters.

**As an agent skill.** The folder layout follows the `SKILL.md` convention supported by several agentic coding assistants: a directory per skill, a YAML frontmatter block with `name` and `description`, and the instructions in the body. Copy the folders you want into your assistant's skills directory:

```bash
cp -r skills/03-web-application-security/* ~/.config/agent-skills/
```

The frontmatter `description` is what the assistant matches against, so it is written as a trigger sentence rather than a title.

## Skill format

Every `SKILL.md` has the same skeleton. It is documented in [docs/skill-format.md](docs/skill-format.md) and enforced loosely — sections that do not apply to a given skill are dropped rather than filled with padding.

```
---
name, domain, description, difficulty, tags, tools
---
## Purpose
## When to use it
## Procedure      <- ordered steps, the part an agent executes
## Cheatsheet     <- commands and flags, the part a human scans
## Reading the output
## Pitfalls
## References
```

## Scope and authorisation

This library covers defensive work and authorised offensive testing: assessments you have written permission to run, CTFs, lab environments, and your own infrastructure. Several domains describe attack techniques, because you cannot detect or fix what you have never seen executed.

What it deliberately does not contain: ready-to-fire exploit payloads for current unpatched software, malware source, techniques whose only purpose is evading defenders, or anything aimed at systems you do not own. Offensive skills are written from the tester's side and always end with the corresponding fix.

Running any of this against infrastructure you are not authorised to test is illegal in most jurisdictions. That is on you, not on the repo.

## Status

Domains are being filled in waves rather than all at once — a skill goes in when it has been used, not when the folder needs a file. `TODO` markers inside a domain README are real, not placeholders for decoration.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Short version: one skill per pull request, keep the frontmatter valid, and no theory without a command that proves it.

## Licence

MIT for the content of this repository. The third-party tools referenced keep their own licences.
