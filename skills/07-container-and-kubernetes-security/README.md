# 07 — Container & Kubernetes Security

Containers don't contain by default. A root process in a privileged pod is a root process on the node. This domain runs from image hygiene up to cluster RBAC — the layers between a vulnerable dependency and a compromised cluster.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [container-image-scanning](01-container-image-scanning/SKILL.md) | Scan images for CVEs and bad practice | ✅ |
| 02 | [dockerfile-hardening](02-dockerfile-hardening/SKILL.md) | Non-root, minimal base, no secrets in layers | ✅ |
| 03 | [kubernetes-rbac-audit](03-kubernetes-rbac-audit/SKILL.md) | Find over-permissive roles and bindings | ✅ |
| 04 | [pod-security-standards](04-pod-security-standards/SKILL.md) | Enforce restricted pod configs | ✅ |
| 05 | [container-escape-vectors](05-container-escape-vectors/SKILL.md) | Privileged, hostPath, and capability escapes | ✅ |
| 06 | [secrets-in-kubernetes](06-secrets-in-kubernetes/SKILL.md) | Stop mounting plaintext secrets | ✅ |
| 07 | [network-policies](07-network-policies/SKILL.md) | Default-deny east-west traffic | ✅ |
| 08 | [admission-control](08-admission-control/SKILL.md) | Gate deploys with OPA/Kyverno | ✅ |
| 09 | [runtime-threat-detection](09-runtime-threat-detection/SKILL.md) | Catch anomalous container behaviour | ✅ |
| 10 | [supply-chain-for-images](10-supply-chain-for-images/SKILL.md) | Sign and verify what you deploy | ✅ |

This domain is complete (10/10). Suggested order: image scanning → Dockerfile hardening → RBAC audit; admission-control is the gate that makes the rest enforceable.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>