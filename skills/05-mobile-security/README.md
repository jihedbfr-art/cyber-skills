# 05 — Mobile Security

The app on the device is a client you don't control. Anything it stores, hardcodes, or trusts is readable by whoever owns the phone. This domain covers static and dynamic assessment of Android and iOS builds, roughly along the OWASP MASVS.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [android-static-analysis](01-android-static-analysis/SKILL.md) | Decompile an APK and read it for secrets and flaws | ✅ |
| 02 | [ios-static-analysis](02-ios-static-analysis/SKILL.md) | Inspect an IPA, entitlements, and binary protections | ✅ |
| 03 | [insecure-data-storage](03-insecure-data-storage/SKILL.md) | Find secrets in prefs, keychains, sqlite, logs | ✅ |
| 04 | [ssl-pinning-bypass](04-ssl-pinning-bypass/SKILL.md) | Defeat pinning in a lab to inspect traffic | ✅ |
| 05 | [dynamic-instrumentation-frida](05-dynamic-instrumentation-frida/SKILL.md) | Hook running apps to observe and change behaviour | ✅ |
| 06 | [mobile-api-traffic](06-mobile-api-traffic/SKILL.md) | Proxy and test the backend the app talks to | ✅ |
| 07 | [deep-link-and-ipc-abuse](07-deep-link-and-ipc-abuse/SKILL.md) | Intent, URL scheme, and IPC surface | ✅ |
| 08 | [reverse-engineering-protections](08-reverse-engineering-protections/SKILL.md) | Root/jailbreak detection, obfuscation, tamper checks | ✅ |
| 09 | [mobile-auth-and-biometrics](09-mobile-auth-and-biometrics/SKILL.md) | Local auth done wrong | ✅ |
| 10 | [play-appstore-hardening](10-play-appstore-hardening/SKILL.md) | Ship-side controls and store requirements | ✅ |

This domain is complete (10/10). `android-static-analysis` is the entry point; the biggest impact is usually in `mobile-api-traffic` (the backend), and `play-appstore-hardening` ties the defensive side together.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>