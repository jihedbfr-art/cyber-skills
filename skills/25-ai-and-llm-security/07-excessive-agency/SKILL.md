---
name: excessive-agency
domain: 25-ai-and-llm-security
description: Use when an LLM system can take actions with real consequences — deciding how much autonomy, permission, and functionality it should have so a bad decision can't cause serious harm.
difficulty: intermediate
tags: [ai, llm, agent, excessive-agency, least-privilege, owasp-llm]
tools: []
---

## Purpose

Excessive agency is what happens when an LLM system is given more capability, permission, or autonomy than the task needs — so when it does something wrong (through a hallucination, a prompt injection, or just a bad inference), the consequences are severe. This skill is the design-side control: bounding what the model is *allowed* to do, so its inevitable mistakes stay cheap. It's the mitigation the prompt-injection and agent-tool-abuse skills keep pointing back to.

## When to use it

Designing or reviewing any LLM system that can act — call functions, trigger workflows, modify data, send messages, spend money. The more the system does automatically, the more this matters. It's a design review as much as a security test.

## The three excesses

OWASP frames excessive agency as too much of any of these:

- **Excessive functionality** — the model has access to tools/functions it doesn't need for its task (a support bot with a `delete_user` tool it never legitimately uses).
- **Excessive permissions** — a tool the model uses runs with more privilege than needed (a "read a record" tool whose DB account can also write and delete).
- **Excessive autonomy** — the model executes consequential actions without human confirmation (it sends the email / makes the payment itself, no approval).

## Procedure

1. **Inventory capabilities.** List every tool/function/integration the system can invoke and the privilege each runs with. This is the review's foundation — the risk is the sum of what it can do.
2. **Challenge each capability against the task.** For every tool, ask "does the model actually need this to do its job?" Remove functionality that isn't required — a tool that isn't there can't be misused.
3. **Right-size permissions.** For each retained tool, check it runs with the minimum privilege — read-only where it only reads, scoped to the relevant data, no broad admin. A stolen or misdirected action then does little.
4. **Assess autonomy per action.** For each consequential action (anything with side effects — send, pay, delete, modify, publish), decide whether the model should do it autonomously or require human approval. High-impact and irreversible actions need a human in the loop, outside the model's control.
5. **Check the confirmation is real.** A confirmation the model can generate for itself (another model turn) is not a control — the approval must be a deterministic gate or a human, so a manipulated model can't self-approve.
6. **Bound and log.** Add rate/spend limits on actions, constrain outputs to allowed operations, and log every action so misuse is visible and reversible where possible.

## Cheatsheet

```
the three excesses (reduce each)
  functionality -> remove tools the task doesn't need
  permissions   -> least privilege on the tools it keeps (read-only where possible)
  autonomy      -> human approval for consequential/irreversible actions

per capability, ask
  [ ] does the task actually require this tool?           (else remove)
  [ ] does it run with minimum privilege / scope?         (else tighten)
  [ ] is this action consequential/irreversible?          (if so, gate it)
  [ ] is the confirmation OUTSIDE the model's control?    (else it's not real)
  [ ] rate/spend limits + logging on the action?

design test: if a prompt injection fully controlled the model, what's the worst
             it could do with the tools/permissions/autonomy it has? -> that's your risk.
```

## Reading the design

- **Tools the model never legitimately uses** = pure downside; they only add attack surface. Remove them.
- **A broadly-privileged tool** (read tool on a read-write-delete DB account) = a misdirected action does far more than intended. Scope it down.
- **Consequential actions executed with no human gate** = one hallucination or injection equals one executed action. Add approval for the high-impact ones.
- **A "confirmation" that's just another model turn** = not a control; a manipulated model approves itself. The gate must be deterministic or human.
- **The worst-case-if-fully-controlled being severe** = the system has excessive agency by definition; reduce capability until that worst case is tolerable.

## The fix

- **Minimise functionality** — grant only the tools the task genuinely needs. The strongest control is simply not giving the capability.
- **Least-privilege permissions** on every tool — read-only, scoped to the necessary data, no standing admin.
- **Human-in-the-loop for consequential and irreversible actions**, with the approval enforced outside the model so it can't be prompted away.
- **Constrain outputs** to an allowlist of operations, and add **rate/spend limits** so even permitted actions can't run away.
- **Log all actions** for auditability and reversibility.
- Design so that the answer to "what could a fully-manipulated model do here?" is "not much" — that's the goal state.

## Pitfalls

- **Granting tools "just in case".** Every unused capability is free attack surface. If the task doesn't need it, don't wire it in.
- **Broad tool permissions for convenience.** A read task on a read-write connection is an unnecessary blast-radius increase.
- **Model-controlled confirmations.** If the "are you sure?" is answerable by the model, a manipulated model self-approves. Put the gate outside.
- **Confusing capable with safe.** More autonomy is more useful and more dangerous; scale it to the reversibility and impact of the actions.

## References

- OWASP Top 10 for LLM Applications — LLM06 Excessive Agency
- OWASP LLM — LLM01 Prompt Injection (the trigger this bounds)
- NIST AI Risk Management Framework
- MITRE ATLAS
