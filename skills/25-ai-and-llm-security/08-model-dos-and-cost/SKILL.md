---
name: model-dos-and-cost
domain: 25-ai-and-llm-security
description: Use when an LLM app could be driven to exhaust resources or run up a large bill through crafted inputs — testing for unbounded consumption and the limits that contain it.
difficulty: intermediate
tags: [ai, llm, dos, cost, resource-consumption, owasp-llm]
tools: []
---

## Purpose

LLM calls are slow and expensive, which makes them a resource-consumption target unlike a normal API. A single crafted request can force the model to generate enormous output, an unbounded agent loop can rack up thousands of calls, and a public LLM endpoint with no limits is a direct line to your compute budget. This skill covers testing an LLM app for denial-of-service and cost-exhaustion, and the controls that bound both.

## When to use it

Any LLM feature exposed to users, especially public-facing ones, agent loops that call the model repeatedly, and anything billed per-token against a provider. The "denial of wallet" angle — where the attack costs you money rather than downtime — is specific to this domain and easy to overlook.

## Procedure

1. **Check for basic request rate limiting** on the LLM endpoint. Without it, an attacker sends unlimited expensive calls — costly and potentially degrading for everyone. Confirm per-user and per-IP limits exist (the API rate-limiting skill's methods apply).
2. **Test output-length amplification.** Can a small prompt force a very large, expensive completion? Ask the model to produce maximum-length output ("write 10,000 words on…", "repeat X many times") and see whether output tokens are capped:
   ```
   "List every integer from 1 to 100000, one per line."
   ```
3. **Test input-size limits.** Very long inputs (huge pasted text, a massive document to summarise) consume context and cost. Is there a cap on input length / document size?
4. **Test agent-loop bounds.** For agentic systems, can the model be pushed into a long or infinite tool-calling loop (each iteration a paid call)? A task that never converges, or a prompt that induces repeated tool use, is a cost/DoS vector — check for a max-iterations/step budget.
5. **Test for recursive/expensive patterns** — prompts that trigger the model to call itself, expand fractally, or chain sub-tasks without bound.
6. **Assess the cost model.** Is spend capped anywhere (per user, per session, globally)? An uncapped pay-per-token endpoint means the attack's cost scales directly to your bill — flag this as financial impact even at modest request volume.

## Cheatsheet

```
consumption vectors to test
  request rate     -> unlimited calls? (per-user + per-IP limit present?)
  output length    -> "generate 10000 words / repeat N times" -> capped?
  input size       -> huge input/document accepted unbounded?
  agent loops      -> can it be driven into long/infinite tool-call loops?
  recursion        -> prompts that fan out into many sub-calls?

the two impacts
  DoS      -> service degraded/unavailable for others
  denial of wallet -> your provider bill scales with the attack (often worse)

controls to verify
  max input tokens, max output tokens, rate limits (user+IP),
  agent step/iteration cap, timeouts, per-user/global spend caps
```

## Reading the output

- **No rate limiting on the LLM endpoint** = both DoS and cost-exhaustion are open; an attacker's script maps directly to your bill and your capacity. High impact.
- **Uncapped output length** = one cheap request produces a maximally expensive completion; a strong amplification factor for an attacker.
- **An agent that can be looped without a step cap** = potentially unbounded paid calls from a single interaction — the most dangerous cost vector in agentic systems.
- **No spend cap anywhere** = "denial of wallet"; even without taking the service down, an attacker runs up real money. Rate it as financial impact, not just availability.
- **Sensible token/rate/step/spend limits present** = the good state; confirm they actually enforce (test past the limit).

## The fix

- **Cap input and output tokens** per request — a hard maximum on completion length kills output amplification, and an input cap bounds context cost.
- **Rate limit** per user and per IP on the LLM endpoint (gateway-enforced), so no single client can flood it.
- **Bound agent loops** with a maximum step/iteration budget and a timeout, so a non-converging task can't spin indefinitely on paid calls.
- **Set spend caps** — per-user quotas and a global budget alarm — so cost can't run away even if a limit is missed. Alert on cost spikes.
- **Validate and limit input size** (document/upload length) before it reaches the model.
- **Use a smaller/cheaper model** for untrusted or high-volume paths where you can, reducing the per-call blast radius.

## Pitfalls

- **Rate limiting requests but not output length.** One allowed request that generates a novel is still expensive. Cap tokens, not just call count.
- **Ignoring denial of wallet.** Teams test for downtime and miss that the real damage is the bill. An uncapped pay-per-token endpoint is a financial vulnerability.
- **Unbounded agent loops.** The convenience of "let the agent keep working until done" becomes unbounded cost when the task never converges or is adversarially steered.
- **No global spend alarm.** Per-user limits help, but a distributed attack or a missed edge case still adds up — a budget alarm is the backstop that tells you before the invoice does.

## References

- OWASP Top 10 for LLM Applications — LLM04 Model Denial of Service / Unbounded Consumption
- OWASP API Security — Unrestricted Resource Consumption (API4)
- Provider rate-limit and quota documentation
- CWE-770 (Allocation of Resources Without Limits)
