# 05 — Mobile Security

The app on the device is a client you don't control. Anything it stores, hardcodes, or trusts is readable by whoever owns the phone. This domain covers static and dynamic assessment of Android and iOS builds, roughly along the OWASP MASVS.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [android-static-analysis](01-android-static-analysis/SKILL.md) | Decompile an APK and read it for secrets and flaws | ✅ |
| 02 | ios-static-analysis | Inspect an IPA, entitlements, and binary protections | TODO |
| 03 | insecure-data-storage | Find secrets in prefs, keychains, sqlite, logs | TODO |
| 04 | ssl-pinning-bypass | Defeat pinning in a lab to inspect traffic | TODO |
| 05 | dynamic-instrumentation-frida | Hook running apps to observe and change behaviour | TODO |
| 06 | mobile-api-traffic | Proxy and test the backend the app talks to | TODO |
| 07 | deep-link-and-ipc-abuse | Intent, URL scheme, and IPC surface | TODO |
| 08 | reverse-engineering-protections | Root/jailbreak detection, obfuscation, tamper checks | TODO |
| 09 | mobile-auth-and-biometrics | Local auth done wrong | TODO |
| 10 | play-appstore-hardening | Ship-side controls and store requirements | TODO |

`android-static-analysis` is the entry point; the rest of the roster is next.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>