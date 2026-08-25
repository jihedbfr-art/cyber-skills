---
format: "v2"
name: "the-pyramid-of-pain"
title: "The Pyramid Of Pain"
title_fr: "La pyramide de la douleur"
description: "Use when deciding which indicators to prioritise — a model for valuing intelligence by how much it costs the attacker to change, so you invest in detections that actually hurt them."
description_fr: "À utiliser pour décider quels indicateurs prioriser — un modèle qui évalue le renseignement selon le coût qu'il impose à l'attaquant pour le changer, afin d'investir dans des détections qui lui font vraiment mal."
domain: "21-threat-intelligence"
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

Not all indicators are equal. Blocking a hash stops exactly one file; the attacker recompiles in seconds and you're back to zero. Detecting their *behaviour* forces them to change how they operate, which is expensive and slow. The Pyramid of Pain is the model for this — it ranks indicator types by how much pain it causes the adversary when you deny them. This skill covers using it to prioritise where intelligence and detection effort goes.

### When to use it

When you're deciding what to do with threat intel — which indicators to block, what detections to build, how to value a feed. It reframes "we have 10,000 IOCs" into "which of these actually cost the attacker anything".

### The pyramid, bottom to top

Effort to detect rises as you climb — and so does the pain you inflict.

- **Hash values** (bottom) — trivial for the attacker to change. Recompile, repack, and the hash is new. Blocking hashes catches only the exact known sample. Lowest pain.
- **IP addresses** — easy to change. Attackers rotate through hosting and proxies cheaply. Blocking IPs is a minor, temporary inconvenience.
- **Domain names** — a bit harder; they cost money and time to register and set up, but still disposable.
- **Network / host artifacts** — patterns in how their tools talk or what they leave on disk (user-agents, URI patterns, registry keys). Now the attacker has to modify tooling to evade you. Real friction begins here.
- **Tools** — the actual utilities they use. Denying these forces them to find or build new tooling. Significant pain.
- **TTPs** (top) — their tactics, techniques, and procedures — *how* they operate. Detect at this level and the attacker has to relearn their craft to get past you. Maximum pain, and the most durable detection.

### Procedure

1. **Classify each indicator** by its level on the pyramid. A feed dump of hashes and IPs is bottom-tier; a report describing an actor's lateral-movement technique is top-tier.
2. **Weight your response by level.** Block the low-tier indicators (cheap to action, cheap for them to change — still worth doing at scale and speed), but invest your *detection engineering* effort higher up.
3. **Push detections upward.** For a given threat, ask: can I detect the behaviour rather than the artifact? A rule for "credential dumping via LSASS access" outlives any hash or IP the actor uses.
4. **Value intel by where it sits.** A report that gives you TTPs is worth more than one that gives you a hash list, because the coverage it enables is durable. Prioritise sources accordingly.
5. **Accept the trade-off.** High-tier detection is harder to build and tune (more false positives, needs richer telemetry) — that cost is the reason it's rare, and the reason it's valuable. Balance quick low-tier wins against durable high-tier investment.

### Cheatsheet

```
PYRAMID OF PAIN                 pain to attacker    your effort
  TTPs                          ^^^^^ maximum       highest (behaviour rules)
  Tools                         ^^^^                high
  Network/Host Artifacts        ^^^                 medium
  Domain Names                  ^^                  low-medium
  IP Addresses                  ^                   low
  Hash Values                   . minimal           trivial

action guide
  low tier (hash/IP/domain): block fast, block at scale — but expect churn
  high tier (artifacts/tools/TTPs): build detections here — durable coverage
  value a feed/report by the highest tier it lets you act on
```

### Reading intel through the pyramid

- **A feed of pure hashes/IPs** is useful for fast blocking but gives no lasting coverage — value it as perishable, and don't mistake volume for value.
- **A report describing an actor's TTPs** (how they gain access, move, persist) is high-value — it enables detections the actor can't cheaply evade. Prioritise extracting these.
- **Your detection set skewed to the bottom of the pyramid** signals fragile coverage — you catch known samples and miss the actor the moment they change a byte. A healthy programme has weight up top.
- **A behaviour-based detection you already have** means an actor switching tools/infra still trips it — that's the durability the pyramid is pointing you toward.

### Applying it (the "fix" / practice)

- **Automate the bottom tier**: ingest and block hashes/IPs/domains at machine speed, but treat them as disposable coverage.
- **Invest human effort at the top**: turn TTP-level intel into behavioural detections (Sigma rules mapped to ATT&CK) — this is where analyst time pays off longest.
- **Map intel to ATT&CK** so TTPs become trackable coverage, not prose in a report.
- **Measure your pyramid balance**: if nearly all detections are hash/IP based, that's a strategic gap to close, not a healthy program.

### Pitfalls

- **Equating IOC count with security.** Ten thousand hashes is ten thousand things an attacker changes for free. Volume at the bottom isn't coverage.
- **Only blocking, never detecting behaviour.** Bottom-tier blocking alone means you're always one recompile behind.
- **Ignoring the bottom entirely.** The top tier is durable but harder; fast low-tier blocking still buys real time and is cheap. Do both, weighted right.
- **Not mapping to ATT&CK.** TTP intel that stays as unstructured text can't drive detection coverage.

### References

- David J. Bianco — The Pyramid of Pain
- MITRE ATT&CK (TTP framework)
- FIRST and SANS threat intelligence guidance

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.