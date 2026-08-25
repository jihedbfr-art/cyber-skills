---
format: "v2"
name: "serverless-security"
title: "Serverless Security"
title_fr: "Sécurité du serverless"
description: "Use when securing serverless functions (Lambda, Cloud Functions, Azure Functions) — over-privileged roles, event-data injection, and the risks that differ from securing servers."
description_fr: "À utiliser pour sécuriser des fonctions serverless (Lambda, Cloud Functions, Azure Functions) — rôles d'exécution trop permissifs, injection via les données d'événement, et les risques propres au serverless qui diffèrent de ceux des serveurs classiques."
domain: "06-cloud-security"
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

Serverless removes the server you'd normally harden, but it doesn't remove the security work — it moves it. The function's IAM role becomes the main attack surface, event data from many sources becomes untrusted input, and short-lived execution changes how you monitor. This skill covers securing serverless functions on their own terms, using AWS Lambda as the concrete example (Cloud Functions and Azure Functions follow the same shape).

### When to use it

Any serverless workload — event-driven functions, API backends on Lambda, glue code triggered by queues/storage events. Especially where functions have broad permissions or process input from external sources.

### Procedure

1. **Right-size the function's execution role — the priority.** Each function runs with an IAM role, and the common failure is one over-broad role shared across functions or granting far more than the function needs. A compromised function is only as dangerous as its role; scope each to the specific actions and resources it uses:
   ```
   aws lambda get-function-configuration --function-name F --query Role
   aws iam list-attached-role-policies --role-name <role>
   ```
2. **Treat every event source as untrusted input.** Functions are triggered by many sources — API Gateway, queues, storage events, streams — and each carries data an attacker may control. The injection classes still apply (SQLi, command injection, SSRF) to whatever the function does with event data. A function that trusts its event payload is as injectable as any web endpoint.
3. **Check dependencies.** Functions ship their dependencies; a vulnerable package in the deployment bundle is your vulnerability. Scan the bundle (ties into the supply-chain and container-scanning skills).
4. **Keep secrets out of the function.** Don't hardcode credentials in code or plain environment variables where practical — pull from a secret manager at runtime. Env vars are visible to anyone who can read the function config.
5. **Bound resource use.** Set sensible timeouts, memory, and concurrency limits — an unbounded or highly-concurrent function is a cost/DoS vector (an attacker floods the trigger), and reserved concurrency caps the blast radius.
6. **Monitor despite ephemerality.** Functions vanish after running, so logging and tracing (to CloudWatch/X-Ray or equivalent) are how you get any visibility — ensure functions log enough to investigate, since there's no host to inspect afterward.

### Cheatsheet

```bash
aws lambda get-function-configuration --function-name F --query Role
aws iam list-attached-role-policies --role-name ROLE
aws iam list-role-policies --role-name ROLE          # inline policies too
  -> scope to least privilege; NO wildcard actions/resources; one role per function

what's different from servers
  attack surface  = the IAM role (compromise = whatever the role allows)
  input           = EVERY event source is untrusted (API GW, SQS, S3, streams)
  secrets         = env vars are visible; use a secret manager
  limits          = timeout / memory / reserved concurrency (cost + DoS control)
  monitoring      = ephemeral -> logging/tracing is your only forensics
```

### Reading the review

- **A broad or wildcard execution role** = a compromised function becomes a wide breach; the role, not the code, defines the damage. The highest-value fix. Scope it per function.
- **A shared role across many functions** = compromise of the least-trusted function grants the union of all their permissions. Separate roles.
- **Event data flowing unsanitised into a query/command/HTTP call** = injection, exactly as in web apps — the event source doesn't make it safe.
- **Secrets in environment variables** = readable by anyone who can view the function config; a leak vector. Move to a secret manager.
- **No timeout/concurrency limits** = an attacker who controls the trigger runs up cost or exhausts concurrency — a serverless DoS/wallet issue.

### The fix

- **Least-privilege execution roles, one per function**, scoped to exact actions and resources — this single practice bounds most serverless risk.
- **Validate and sanitise event data** as untrusted input; apply the injection defences per sink regardless of which trigger delivered it.
- **Pull secrets at runtime** from a secret manager; keep them out of code and plain env vars.
- **Scan dependencies** in the deployment bundle and keep them updated.
- **Set timeouts, memory, and reserved concurrency** to bound cost and blast radius.
- **Log and trace** enough to investigate after the function is gone; alert on anomalies (unusual invocation spikes, errors).

### Pitfalls

- **Over-privileged / shared execution roles.** The dominant serverless mistake — the role is the real attack surface, and a broad one turns a small compromise into a big one. Least privilege, per function.
- **Trusting event data.** "It's just a queue message / storage event" — an attacker who can influence the source injects through it. Treat all event input as untrusted.
- **Secrets in env vars.** Convenient and visible. Use a secret manager.
- **Ignoring dependencies.** The function bundles them; their vulnerabilities are yours.
- **No monitoring.** Ephemeral execution means no host to inspect later — without logging, an incident is invisible.

### References

- OWASP Serverless Top 10
- AWS Lambda security best practices / Well-Architected serverless
- CWE-269 (privilege management), CWE-74 (injection)
- Cloud provider function security documentation (Lambda, Cloud Functions, Azure Functions)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.