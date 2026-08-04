# 07 — Container & Kubernetes Security

Containers don't contain by default. A root process in a privileged pod is a root process on the node. This domain runs from image hygiene up to cluster RBAC — the layers between a vulnerable dependency and a compromised cluster.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [container-image-scanning](01-container-image-scanning/SKILL.md) | Scan images for CVEs and bad practice | ✅ |
| 02 | dockerfile-hardening | Non-root, minimal base, no secrets in layers | TODO |
| 03 | kubernetes-rbac-audit | Find over-permissive roles and bindings | TODO |
| 04 | pod-security-standards | Enforce restricted pod configs | TODO |
| 05 | container-escape-vectors | Privileged, hostPath, and capability escapes | TODO |
| 06 | secrets-in-kubernetes | Stop mounting plaintext secrets | TODO |
| 07 | network-policies | Default-deny east-west traffic | TODO |
| 08 | admission-control | Gate deploys with OPA/Kyverno | TODO |
| 09 | runtime-threat-detection | Catch anomalous container behaviour | TODO |
| 10 | supply-chain-for-images | Sign and verify what you deploy | TODO |

Suggested order: image scanning (done) → Dockerfile hardening → RBAC audit.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>