---
format: "v2"
name: "android-static-analysis"
title: "Android Static Analysis"
title_fr: "Analyse statique Android"
description: "Use when you have an Android APK and want to read it for secrets, weak configuration, and vulnerable code without running it — the first pass on any mobile assessment."
description_fr: "À utiliser quand vous disposez d'un APK Android et voulez l'examiner à froid pour repérer secrets codés en dur, configuration faible et code vulnérable, sans exécuter l'application — le premier réflexe de toute évaluation mobile."
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

An APK is a zip file, and everything the app ships with is inside it: hardcoded keys, API endpoints, debug flags, and the compiled logic. Static analysis reads all of that without executing the app. This skill covers taking an APK apart, knowing what to look for, and where the real findings hide.

### When to use it

The opening move on any Android assessment, before dynamic testing. It's fast, needs no device, and usually surfaces the first few findings (leaked secrets, insecure settings) on its own.

Only analyse apps you're authorised to test or that you own. Decompiling third-party apps may breach their terms.

### Procedure

1. Unpack the APK to get the manifest, resources, and smali. `apktool` gives you the decoded `AndroidManifest.xml` and resources:
   ```
   apktool d app.apk -o app-src
   ```
2. Read the **manifest** first — it's the app's security posture in one file:
   - `android:debuggable="true"` — debug build shipped to prod, a finding.
   - `android:allowBackup="true"` — app data extractable via adb backup.
   - exported `activity`/`service`/`receiver`/`provider` with no permission — IPC attack surface.
   - `usesCleartextTraffic="true"` or a permissive network security config — traffic not forced to TLS.
3. Decompile to readable Java to review the logic:
   ```
   jadx -d app-java app.apk
   ```
4. Grep the decompiled source and resources for **secrets and endpoints** — this is where quick wins live:
   ```
   grep -rEi 'api[_-]?key|secret|password|token|BEGIN.*PRIVATE|http://|amazonaws' app-java app-src
   ```
5. Check `res/xml/network_security_config.xml` and `strings.xml` for base URLs, disabled cert validation, and trust-all configs.
6. For breadth, run an automated pass to catch what manual review misses and to score the app against a checklist:
   ```
   mobsf   # upload the APK to the local MobSF instance
   ```
7. Triage findings: a live secret or cleartext prod endpoint outranks a theoretical exported component with no useful function behind it.

### Cheatsheet

```bash
apktool d app.apk -o src              # decode manifest + resources + smali
jadx -d out app.apk                   # decompile to Java
unzip -l app.apk                       # peek at contents without decoding

android:debuggable="true"
android:allowBackup="true"
android:exported="true"   (with no permission)
android:usesCleartextTraffic="true"

grep -rEi 'api_key|apikey|secret|password|token|http://|s3\.amazonaws' out
```

### Reading the output

- **A live API key or credential in the source** is a direct finding — the app ships it to every user's device, so it's public.
- **`debuggable="true"` in a release build** lets anyone attach a debugger and inspect/modify the running app.
- **Cleartext traffic allowed or cert validation disabled** means the app's backend calls can be intercepted — feeds the mobile-API-traffic skill.
- **Exported components without permissions** are IPC entry points other apps can invoke; confirm what they do before rating severity.
- **MobSF's high/warning items** are leads, not verdicts — verify each in the decompiled code before reporting.

### The fix

- **Never ship secrets in the APK.** Anything on the device is extractable; move keys server-side and have the app fetch short-lived tokens. Obfuscation slows this down, it doesn't prevent it.
- Set `debuggable="false"` and `allowBackup="false"` for release builds.
- **Force TLS**: remove cleartext traffic, ship a strict network security config, and don't disable certificate validation (pin where appropriate).
- Export only the components that must be, and guard them with permissions and input validation.
- Enable code shrinking/obfuscation (R8/ProGuard) as defence in depth — it raises the cost of reverse engineering without being a security control on its own.

### Pitfalls

- **Treating obfuscation as protection.** It slows analysis; it doesn't hide a hardcoded key from `strings` and grep.
- **Reading Java but skipping the manifest.** The fastest findings (debuggable, backup, cleartext, exports) are all in the manifest.
- **Trusting the automated score.** MobSF flags possibilities; a "high" can be a false positive and a real bug can score low. Verify in source.
- **Ignoring native libraries.** Secrets and logic in `lib/*.so` won't show in the Java decompile — check those separately if the Java layer looks too clean.

### References

- OWASP Mobile Application Security Testing Guide (MASTG)
- OWASP MASVS (verification standard)
- jadx, apktool, and MobSF documentation

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.