# 25 — AI & LLM Security

The newest surface, and the one moving fastest. When an application hands a language model untrusted text and lets it call tools, the text becomes an instruction channel. This domain covers the attacks specific to LLM apps and agents, along the lines of the OWASP LLM Top 10.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [prompt-injection-testing](01-prompt-injection-testing/SKILL.md) | Test direct and indirect prompt injection, and the mitigations | ✅ |
| 02 | [insecure-output-handling](02-insecure-output-handling/SKILL.md) | Treat model output as untrusted before it hits a sink | ✅ |
| 03 | training-data-and-model-supply-chain | Vet models and datasets you didn't build | TODO |
| 04 | sensitive-data-leakage | Stop the model from spilling secrets and PII | TODO |
| 05 | [agent-tool-abuse](05-agent-tool-abuse/SKILL.md) | Contain what an agent's tools can actually do | ✅ |
| 06 | rag-security | Poisoning and access control in retrieval | TODO |
| 07 | excessive-agency | Limit autonomy and require confirmation on side effects | TODO |
| 08 | model-dos-and-cost | Prompt-driven resource and cost attacks | TODO |
| 09 | jailbreak-resistance-testing | Evaluate guardrails honestly | TODO |
| 10 | ml-model-security | Adversarial inputs and model theft beyond LLMs | TODO |

Start with `prompt-injection-testing` — it's the root cause behind most of the other entries.
