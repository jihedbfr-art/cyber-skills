---
format: "v2"
name: "play-appstore-hardening"
title: "Play Appstore Hardening"
title_fr: "Durcissement pour Play Store et App Store"
description: "Use when hardening a mobile app for release — the ship-side controls, store security requirements, and configuration that reduce risk before and after publishing to the app stores."
description_fr: "À utiliser pour durcir une application mobile avant sa mise en production — les contrôles à appliquer côté publication, les exigences de sécurité des stores et la configuration qui réduisent le risque avant et après la publication sur les stores."
domain: "05-mobile-security"
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

The other mobile skills find and exploit weaknesses; this one is the defensive summary — the controls to apply *before* shipping an app, and the store requirements to meet. It pulls together the ship-side hardening (secure configuration, protecting data and traffic, meeting Play/App Store security requirements) into a release checklist, so an app is hardened by the time it's published rather than assessed as vulnerable afterward. It's the "make it secure before release" counterpart to the domain's testing skills.

### When to use it

Preparing a mobile app for release, or reviewing an app's security posture holistically. It ties the domain together — each earlier skill's fix becomes a line in this hardening checklist.

### Procedure

1. **Protect data at rest and in transit** (the storage and traffic skills):
   - Secrets in the secure store (Keystore/Keychain), not plain preferences/plists; minimise stored sensitive data; exclude it from backups.
   - Enforce TLS (no cleartext — Android network security config, iOS ATS with no `NSAllowsArbitraryLoads`); consider certificate pinning for high-value apps.
2. **Secure the configuration** (the static-analysis skills):
   - No hardcoded secrets in the app (they ship to every device); no debuggable release builds; minimal exported components with explicit exported status; validated deep-link input.
3. **Enforce security server-side, not just in the app** — the recurring principle: client-side controls (biometrics, validation, anti-tampering) are bypassable, so authentication, authorization, and sensitive logic must be enforced on the backend. The app is a client; don't trust it.
4. **Meet the app store security requirements.** Google Play and the App Store enforce security policies (data safety declarations, permission justification, target API levels, prohibited behaviours, SDK requirements). Meeting these is mandatory for publishing and catches some baseline issues — but store review is not a security assessment; passing it isn't proof of security.
5. **Minimise permissions and SDKs.** Request only the permissions the app needs (over-permissioned apps are risk and a store-review flag), and vet third-party SDKs — a malicious or vulnerable SDK is your app's vulnerability (the supply-chain angle, on mobile).
6. **Add resilience controls proportionate to the threat** (the RE-protections skill) — root/jailbreak detection, obfuscation, anti-tampering for high-value apps, understanding they raise cost but aren't absolute and don't replace server-side security.
7. **Test before shipping.** Run the app through the assessment skills (static, storage, traffic, IPC, auth) and an automated pass (MobSF) as a pre-release gate, so vulnerabilities are fixed before publishing, not found by users or attackers after.
8. **Plan for post-release.** Have an update mechanism to push fixes, monitor for issues, and watch for repackaged/malicious clones of your app in stores.

### Cheatsheet

```
ship-side hardening checklist (each = an earlier skill's fix)

DATA
  secrets in Keystore/Keychain (not prefs/plist) ; minimise stored ; exclude from backups
  TLS enforced (no cleartext: Android network-security-config / iOS ATS, no NSAllowsArbitraryLoads)
    ; pinning for high-value apps
CONFIG
  no hardcoded secrets (ship to every device) ; no debuggable release ; minimal exported components
    (explicit exported status) ; validate deep-link input
SERVER-SIDE (recurring principle)
  client controls (biometrics/validation/anti-tamper) BYPASSABLE -> auth/authz/sensitive logic = BACKEND
STORE REQUIREMENTS
  Play/App Store security policies (data safety, permission justification, target API, SDK reqs) — mandatory
  BUT store review != security assessment (passing != secure)
PERMISSIONS/SDKs: request only needed ; VET third-party SDKs (malicious/vuln SDK = your vuln)
RESILIENCE (proportionate): root/JB detection, obfuscation, anti-tamper (raise cost, not absolute)
TEST before ship: assessment skills + MobSF as pre-release GATE (fix before publish)
POST-RELEASE: update mechanism, monitor, watch for repackaged/clone apps
```

### Reading the readiness

- **Secrets in the secure store, TLS enforced, no hardcoded secrets, minimal exported surface** = the data/config baseline met; the most common mobile findings pre-empted before release.
- **Security enforced server-side** = the app doesn't trust the client; even a fully bypassed app (root, hooked, repackaged) can't reach what the backend independently gates. The single most important principle — client-side controls are bypassable.
- **Meeting store requirements but treating that as "secure"** = a false sense of security; store review checks policy compliance, not a real assessment. Passing it isn't proof of security — still run the assessment skills.
- **Over-permissioned or unvetted third-party SDKs** = risk and a store flag; a malicious/vulnerable SDK is your app's vulnerability. Minimise permissions and vet SDKs.
- **Resilience controls proportionate to a high-value app** = appropriate defense-in-depth; but not a substitute for server-side security. Don't over-rely on them.
- **A pre-release assessment (static/storage/traffic/IPC/auth + MobSF) passed** = vulnerabilities fixed before users/attackers find them — the point of ship-side hardening.

### Pitfalls

- **Assessing security after release, not before.** The point is to ship hardened; run the assessment skills as a pre-release gate so users and attackers don't find the vulnerabilities first.
- **Trusting the client.** The recurring mobile mistake — client-side controls (biometrics, validation, anti-tampering) are bypassable, so auth/authz/sensitive logic must be server-side. Never rely on the client for security.
- **Treating store review as a security assessment.** Play/App Store review checks policy, not security; passing it isn't proof the app is secure. Do your own testing.
- **Hardcoded secrets and debuggable release builds.** Secrets ship to every device; debuggable builds expose the app. Catch these in static analysis before release.
- **Over-permissioning and unvetted SDKs.** Both are risk (and store flags); minimise permissions and vet third-party SDKs, which are your app's supply chain.
- **Over-relying on resilience controls.** Root detection and obfuscation raise cost but are bypassable; they complement server-side security, not replace it.

### References

- OWASP MASVS (the mobile security verification standard — the hardening checklist) and MASTG
- Google Play and Apple App Store security/policy requirements
- The other mobile skills (each fix is a hardening item) and MobSF (pre-release scanning)
- Android network security config and iOS App Transport Security documentation

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.