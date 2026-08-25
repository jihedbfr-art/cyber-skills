---
format: "v2"
name: "training-data-and-model-supply-chain"
title: "Training Data And Model Supply Chain"
title_fr: "Données d'entraînement et chaîne d'approvisionnement des modèles"
description: "Use when your app depends on models or datasets you didn't build — vetting them for poisoning, backdoors, and malicious serialization before you trust them in production."
description_fr: "À utiliser quand votre application dépend de modèles ou de jeux de données que vous n'avez pas construits — les évaluer contre l'empoisonnement, les backdoors et la sérialisation malveillante avant de leur faire confiance en production."
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

Most teams don't train models from scratch — they pull a pretrained model off a hub and fine-tune, or use a third-party dataset. That's a supply chain, with the same risks as software dependencies plus a few of its own: a poisoned dataset that plants a backdoor, a model file that runs code when you load it, or a tampered model uploaded under a trusted name. This skill covers vetting what you didn't build before it reaches production.

### When to use it

Any time you adopt an external model, dataset, or embedding — a Hugging Face checkpoint, a public dataset, a base model for fine-tuning. It's the ML-specific extension of the software supply-chain domain.

### Procedure

1. **Verify provenance and integrity.** Where did the model/dataset actually come from? Prefer official/verified publishers, pin to a specific commit/revision (not a moving tag), and verify checksums so you get the exact artifact you reviewed — not one swapped later.
2. **Beware malicious serialization — the immediate code-execution risk.** Model files in Python **pickle** format (and some others) can execute arbitrary code *on load*. Loading an untrusted `.pkl`/`.bin`/`.pt` can run code before the model does anything. Scan before loading, and prefer safe formats:
   ```
   picklescan --path model.bin      # flag dangerous opcodes / imports
   # prefer safetensors, which can't carry executable payloads
   ```
3. **Consider poisoning and backdoors.** A model can be trained (or fine-tuned) to behave normally except on a specific trigger, where it misclassifies or emits attacker-chosen output. You usually can't fully detect this by inspection — so weight trust toward reputable sources, and **evaluate behaviour** on your own held-out and adversarial test set before deploying.
4. **Vet datasets** used for training/fine-tuning: source, licence, and whether they could contain planted poison samples or sensitive/PII data you shouldn't ingest. Untrusted data upstream becomes model behaviour downstream.
5. **Pin and inventory.** Record exactly which model/dataset versions you use (an SBOM-equivalent for ML) so that when a supply-chain issue surfaces, you can answer "are we affected".
6. **Isolate loading.** Load new/untrusted models in a sandboxed environment first, given the code-execution risk, before bringing them near production.

### Cheatsheet

```
before trusting an external model/dataset
  [ ] provenance: verified publisher? pinned to a specific revision/commit?
  [ ] integrity: checksum verified against what you reviewed?
  [ ] format: prefer SAFETENSORS; scan pickle-based files before load
        picklescan --path model.bin   (dangerous opcodes = do NOT load)
  [ ] behaviour: evaluate on your held-out + adversarial test set
  [ ] dataset: source, licence, PII, poisoning potential
  [ ] inventory: pin versions (ML SBOM) for later "are we affected?"
  [ ] load untrusted models in a sandbox first (code-exec on load)

key risk: pickle deserialization = arbitrary code execution ON LOAD
```

### Reading the review

- **A pickle-format model from an unverified source** = potential code execution the moment you load it — the most immediate, concrete risk here. Scan it, sandbox it, or prefer a safetensors version.
- **A moving tag (`latest`, `main`) instead of a pinned revision** = you can't guarantee the artifact you reviewed is the one you'll load next deploy. Pin it.
- **An unverified publisher / typosquatted model name** = supply-chain impersonation, exactly like a malicious package. Confirm the real source.
- **Odd behaviour on specific trigger inputs** during evaluation = a possible backdoor; investigate before trusting, and lean on provenance since backdoors resist inspection.
- **A dataset of unknown provenance** = poisoning and PII risk that becomes baked into your model — vet upstream, because you can't easily un-learn it.

### The fix / best practice

- **Prefer safe formats** (safetensors) that can't carry executable payloads; **scan** any pickle-based artifact and load untrusted ones only in isolation.
- **Pin to verified publishers and specific revisions**, verify checksums, and keep an inventory of model/dataset versions.
- **Evaluate behaviour** on your own tests (including adversarial/trigger probes) before production — provenance plus testing is your best defence against poisoning you can't directly detect.
- **Vet datasets** for source, licence, PII, and poisoning risk before training on them.
- Treat model updates like dependency updates: reviewed, pinned, and reversible.

### Pitfalls

- **Loading pickle models from the internet.** It's remote code execution waiting to happen. Scan and sandbox, or use safetensors.
- **Trusting a name.** Typosquatted or impersonated model repos exist; verify the publisher and integrity, not just the label.
- **Assuming you can inspect away a backdoor.** You mostly can't — provenance and behavioural evaluation matter more than staring at weights.
- **No version pinning.** A moving tag means you can't reproduce or audit what you actually shipped, and can't answer "are we affected".

### References

- OWASP Top 10 for LLM Applications — LLM03 Training Data Poisoning, LLM05 Supply Chain
- Hugging Face security documentation (safetensors, malware scanning)
- picklescan and model-scanning tooling
- NIST AI 100-2 (adversarial ML: poisoning)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.