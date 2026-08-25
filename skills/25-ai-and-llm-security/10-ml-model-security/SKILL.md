---
format: "v2"
name: "ml-model-security"
title: "Ml Model Security"
title_fr: "Sécurité des modèles ML classiques"
description: "Use when securing traditional ML models (classifiers, detectors, recommenders) rather than LLMs — adversarial examples, model theft, and inference attacks, plus defences."
description_fr: "À utiliser pour sécuriser des modèles ML traditionnels (classifieurs, détecteurs, recommandeurs) plutôt que des LLM — exemples adverses, vol de modèle et attaques par inférence, ainsi que les défenses associées."
domain: "25-ai-and-llm-security"
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

Not all AI is an LLM. Fraud detectors, image classifiers, spam filters, and recommenders are classical ML models with their own attack surface — one that predates the LLM hype and still matters, especially when a model gates a security decision. This skill covers the main attack classes against predictive ML models and the defences, so the rest of this domain isn't only about language models.

### When to use it

Any system where a trained ML model makes or informs a decision: fraud/abuse detection, malware or spam classification, content moderation, biometric matching, recommendation. Especially where an attacker benefits from fooling the model (evading a detector) or from stealing it.

### The attack classes

- **Adversarial examples (evasion)** — an input crafted with small, often imperceptible perturbations that flips the model's decision. The malware that a classifier now sees as benign; the image tweaked to dodge a filter. This is the headline ML attack and directly undermines security models.
- **Model extraction / theft** — querying a model enough to reconstruct a functional copy, stealing the intellectual property and, worse, enabling offline attack development against the stolen copy.
- **Membership inference** — determining whether a specific record was in the training data, a privacy breach (e.g. proving someone's data was used).
- **Model inversion** — reconstructing sensitive training inputs from model outputs.
- **Data poisoning** — corrupting the training data so the model learns attacker-chosen behaviour (covered from the supply-chain angle in the training-data skill; here it's the model-behaviour impact).

### Procedure

1. **Identify what the model decides and who benefits from fooling it.** A model that gates a security decision (fraud, malware, moderation) is a high-value evasion target; a recommender is a lower-stakes one. This frames which attacks matter.
2. **Test evasion (adversarial robustness)** where the model is a security control. Can small, plausible perturbations to an input flip the decision? For a detector, the question is whether an attacker can reliably craft a malicious input the model passes. Evaluate robustness with adversarial test inputs, not just clean accuracy.
3. **Assess extraction exposure.** Is the model queryable at scale (a public prediction API)? Estimate how many queries would reveal enough to clone it, and whether outputs (full probability vectors vs just a label) leak more than necessary.
4. **Assess inference/privacy risk** if the model was trained on sensitive data — could an attacker infer membership or reconstruct inputs? Higher risk with overfit models and rich output.
5. **Check the training pipeline** for poisoning exposure (who can influence training data) — ties into the supply-chain skill.
6. **Weigh consequences.** A fooled fraud model that lets transactions through has direct financial impact; the severity comes from what the decision controls, not the elegance of the attack.

### Cheatsheet

```
attack               what it does                        defence direction
-------------------  ----------------------------------  -----------------------------
adversarial example  perturb input to flip decision      adversarial training, input
  (evasion)          (evade a detector/filter)             checks, ensembles, don't rely
                                                            on one model for security
model extraction     clone the model via many queries    rate-limit queries, return
                                                            labels not full probabilities,
                                                            watermark, monitor
membership inference infer if a record was in training   limit overfit, differential
                                                            privacy, reduce output detail
model inversion      reconstruct training inputs          same as above
data poisoning       corrupt training -> learned backdoor vet/validate training data

frame by: what does the model DECIDE, and who gains from fooling it?
```

### Reading the assessment

- **A security-gating model that's fooled by small perturbations** = the control is evadable; an attacker crafts inputs that pass. High impact, because the model was the defence.
- **A public prediction API returning full probability vectors** = easier extraction and inference; the richer the output, the more it leaks. Often a return-less-detail fix.
- **An overfit model on sensitive training data** = membership-inference and inversion risk — a privacy exposure, not just a security one.
- **A model whose decision has high consequence** (money, access, moderation) = rate any evasion finding by that consequence, and question whether a single ML model should be the sole gate.
- **Robust evaluation showing stable decisions under adversarial inputs** = the good state, but ML robustness is relative — treat it as raised difficulty, not immunity.

### The defences

- **Don't let a single ML model be the sole security control.** Combine it with deterministic rules and defence in depth, so evading the model isn't game over. This is the most important architectural point — models are one signal, not the whole decision.
- **Adversarial training and robustness evaluation** — train on adversarial examples and measure robustness, not just clean accuracy, for models used as controls.
- **Protect the query interface** against extraction: rate-limit, return minimal output (labels over full probabilities), monitor for scraping patterns, and consider watermarking.
- **Reduce privacy leakage**: limit overfitting, consider differential privacy for sensitive training data, and minimise output detail.
- **Secure the training pipeline** against poisoning (vet data sources, validate inputs) — see the supply-chain skill.
- **Monitor production** for distribution shift and anomalous query patterns that signal an attack in progress.

### Pitfalls

- **Trusting a model as a standalone security gate.** Evasion attacks exist for essentially every model; make it one layer, not the only one.
- **Measuring only clean accuracy.** A model with great accuracy on normal inputs can be trivially evadable — robustness against adversarial inputs is the security-relevant metric.
- **Exposing full probability outputs publicly.** They accelerate extraction and inference; return the minimum a legitimate consumer needs.
- **Ignoring classical ML because "AI = LLMs now".** Predictive models still make high-stakes decisions and remain a live attack surface.

### References

- NIST AI 100-2 (Adversarial Machine Learning: A Taxonomy and Terminology)
- MITRE ATLAS (adversarial threat landscape for AI systems)
- OWASP Machine Learning Security Top 10
- Academic literature on adversarial examples, model extraction, and membership inference

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.