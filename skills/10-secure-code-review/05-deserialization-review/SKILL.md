---
format: "v2"
name: "deserialization-review"
title: "Deserialization Review"
title_fr: "Revue de la désérialisation"
description: "Use when reviewing code that turns bytes back into objects — spotting the unsafe deserialisation paths that lead to RCE, across Java, Python, .NET, and friends."
description_fr: "À utiliser lors de la revue de code qui reconstruit des objets à partir d'octets — pour repérer les chemins de désérialisation non sûrs menant à une exécution de code à distance, en Java, Python, .NET et au-delà."
domain: "10-secure-code-review"
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

Unsafe deserialisation is one of the few review findings that jumps straight to remote code execution. The pattern is deceptively small: an app takes bytes from somewhere untrusted and reconstructs a live object graph from them, and the reconstruction process itself can be steered into running code. This skill is about recognising the native serialisers and misused parsers that make that possible.

### When to use it

Reviewing anything that reads objects from an untrusted boundary: request bodies, message queues, caches, cookies, uploaded files, inter-service calls. The dangerous version is native/binary object serialisation; plain JSON *data* is usually fine — until a library is told to instantiate arbitrary types from it.

### The dangerous calls by ecosystem

- **Java** — `ObjectInputStream.readObject()` on attacker-reachable bytes. This is the classic gadget-chain RCE (the whole `ysoserial` world). Also: Jackson/`ObjectMapper` with **default typing enabled** (`enableDefaultTyping()`, `@JsonTypeInfo` on `Object`), XStream, SnakeYAML `new Yaml().load(...)` on untrusted input, `XMLDecoder`.
- **Python** — `pickle.loads`, `cPickle`, `shelve`, `dill`, `yaml.load` without `SafeLoader`, `jsonpickle`. `pickle` on untrusted data is game over by design.
- **.NET** — `BinaryFormatter` (deprecated for exactly this reason), `LosFormatter`, `NetDataContractSerializer`, `TypeNameHandling.All` in Json.NET, `ObjectStateFormatter`.
- **PHP** — `unserialize()` on user input (object injection via magic methods), plus phar-based paths.
- **Ruby** — `Marshal.load`, unsafe `YAML.load`.

The common thread: the deserialiser is allowed to **decide which types to instantiate** based on the incoming data. That type-name control is the vulnerability, more than the parsing itself.

I've hit the Jackson one on a real Spring service — `enableDefaultTyping()` had been switched on years earlier to make a polymorphic field "just work," and it quietly turned every JSON endpoint into a type-instantiation primitive. It looked like ordinary JSON handling. Grep for the typing config, not just for `readObject`.

### Procedure

1. Grep the native serialisers and the "polymorphic typing" switches (cheatsheet).
2. For each, trace the input to the boundary: does the byte stream come from anywhere a user, a queue, or another service can influence? Internal-only, trusted-source data lowers the severity but rarely to zero.
3. If it's native object serialisation on untrusted input, it's a finding — you don't need to build the gadget chain to call it, presence is enough.
4. For JSON/YAML, the question is narrower: **is type information taken from the payload?** Default typing, `TypeNameHandling`, non-Safe YAML loaders = yes = finding. Plain field-mapping to a fixed DTO = fine.
5. Check for after-the-fact "fixes" like class blocklists — note them as weak (blocklists get bypassed by new gadgets) and recommend the real fix.

### Cheatsheet

```bash
rg -n 'readObject|ObjectInputStream|pickle\.loads|BinaryFormatter|Marshal\.load|unserialize\('
rg -n 'yaml\.load\((?!.*Safe)|new Yaml\(\)\.load|YAML\.load'
rg -n 'enableDefaultTyping|activateDefaultTyping|@JsonTypeInfo|TypeNameHandling'
rg -n 'XMLDecoder|XStream'
```

### Reading it

- **`readObject` / `pickle.loads` / `BinaryFormatter` on untrusted bytes** → treat as RCE. Highest severity in this domain.
- **Jackson/Json.NET with default typing on** → RCE-class even though it "looks like JSON." Flag the config line.
- **`yaml.load` with `SafeLoader` / `safe_load`** → fine; that's the correct form.
- **JSON mapped to a concrete DTO, no type info in the payload** → safe; don't flag.
- **A class allow/blocklist bolted on** → allowlist is defensible, blocklist is a stopgap. Say which it is.

### The fix

Don't deserialise untrusted data into arbitrary types. Prefer a data-only format (plain JSON) mapped to known DTOs; use `SafeLoader`, disable default/polymorphic typing, and retire `BinaryFormatter`/`readObject` on any external boundary. Where polymorphism is genuinely needed, constrain it to an explicit allowlist of permitted types. Integrity-protecting the serialised blob (signing it) helps only when the source is meant to be trusted, and never justifies native deserialisation of user input.

### Pitfalls

- **"It's just JSON, it's safe."** Not with default typing on. That's the sneakiest version.
- **Trusting a class blocklist.** New gadget chains appear; blocklists lag. Allowlist or redesign.
- **Only checking HTTP bodies.** Queues, caches, and cookies deserialise too, and get less scrutiny.
- **Chasing a working exploit before reporting.** Reachable native deserialisation of untrusted data is the finding on its own.

### References

- OWASP Deserialization Cheat Sheet
- CWE-502 (Deserialization of Untrusted Data)
- `ysoserial` (Java) and `ysoserial.net` as background on why presence = risk

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.