---
format: "v2"
name: "dynamic-instrumentation-frida"
title: "Dynamic Instrumentation Frida"
title_fr: "Instrumentation dynamique avec Frida"
description: "Use when analysing a running mobile app by hooking its functions with Frida — observing and modifying behaviour at runtime to test logic, bypass checks, and inspect data."
description_fr: "À utiliser pour analyser une application mobile en cours d'exécution en hookant ses fonctions avec Frida — observer et modifier son comportement à l'exécution pour tester la logique métier, contourner des contrôles et inspecter les données manipulées."
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

Static analysis reads the app; dynamic instrumentation lets you reach into it while it runs — hooking functions to see their arguments and return values, and modifying behaviour to test how the app responds. Frida is the standard tool: it injects into a running app and lets you intercept and change any function call. This skill covers using Frida to analyse mobile app behaviour at runtime, the technique behind pinning bypass, security-control bypass, and observing what the app actually does with data.

### When to use it

The dynamic phase of a mobile assessment, when static analysis has shown you the app's structure and you want to observe or manipulate runtime behaviour. It requires a device you control (rooted/jailbroken is easiest) and an app you're authorised to test.

### Procedure

1. **Set up Frida.** Install the Frida server on the test device (rooted/jailbroken) and the Frida client on your machine; confirm it can attach to the target app:
   ```
   frida-ps -U            # list processes on the USB device
   frida -U -f <package>  # spawn and attach to the app
   ```
   Objection is a Frida-based toolkit that wraps common tasks (pinning bypass, storage inspection) in ready commands — a good starting point.
2. **Hook functions to observe behaviour.** Intercept a function to log its arguments and return value — this reveals what the app does with inputs, what it sends, and how it processes data. Hooking is the core technique:
   ```
   // Frida script: hook a method, log args + return
   Java.perform(() => {
     const C = Java.use('com.app.Crypto');
     C.encrypt.implementation = function (data) {
       console.log('encrypt input:', data);
       const r = this.encrypt(data);
       console.log('encrypt output:', r);
       return r;
     };
   });
   ```
3. **Modify behaviour to test controls.** Change a function's return value or logic at runtime to bypass a check — root detection returning false, a license check passing, a pinning validation succeeding (the pinning skill). This tests whether security controls can be bypassed by an attacker with device control.
4. **Inspect data and internal state.** Read the app's runtime data — decrypted values, keys in memory, tokens — that static analysis or storage inspection can't reach because they only exist at runtime.
5. **Use Objection for common tasks.** For standard operations (SSL pinning bypass, root-detection bypass, keychain/storage dumping, method enumeration), Objection provides them without writing Frida scripts — faster for routine assessment.
6. **Combine with the other techniques.** Frida is the engine behind pinning bypass and RE-protection bypass; use it to enable traffic inspection and to test the anti-tampering controls.
7. **Recognise when the app fights back.** Anti-instrumentation and Frida-detection (part of RE protections) may detect or block Frida; robust apps require bypassing those first.

### Cheatsheet

```
static = read the app ; Frida = reach INTO it while running (hook functions -> observe + MODIFY)
  the engine behind pinning bypass, control bypass, runtime data inspection
  needs: device you control (rooted/JB easiest) + authorised app

setup: frida-server on device ; frida-ps -U (list) ; frida -U -f <pkg> (spawn+attach)
  Objection = Frida toolkit wrapping common tasks (pinning/root bypass, storage dump) — good start

core techniques
  HOOK to observe: log a function's args + return -> what the app does with data/inputs/sends
  MODIFY to test controls: change return/logic -> bypass root detection / license / pinning
    (tests whether an attacker with device control bypasses the control)
  INSPECT runtime state: decrypted values, keys in memory, tokens (static/storage can't reach)
  OBJECTION for routine: pinning bypass, root bypass, keychain/storage dump, method enum

app fights back: anti-instrumentation / Frida-detection (RE-protections) -> bypass first
```

### Reading the analysis

- **A hooked function revealing its arguments and return** = you see what the app actually does with data at runtime — the input to a crypto call, the token it sends, the value of a check. This is Frida's core value: observing behaviour static analysis can only guess at.
- **A security control bypassed by modifying a return value** (root detection → false, pinning → success) = demonstrates the control is bypassable by an attacker with device control; important for judging whether client-side controls are relied upon inappropriately (they shouldn't be the only defence).
- **Runtime data inspected** (decrypted secrets, in-memory keys) = data that only exists while running, invisible to static/storage analysis; Frida reaches it.
- **Objection handling a routine task** (pinning/root bypass, storage dump) = faster than a custom script for standard operations; use it for the common cases.
- **The app detecting/blocking Frida** = anti-instrumentation is present (an RE protection); robust apps require bypassing that before Frida works. Recognise the resistance.
- **Frida enabling traffic inspection and control testing** = the dynamic phase working; it's the enabler for much of mobile assessment.

### Pitfalls

- **Instrumenting an app you're not authorised to test.** Frida modifies a running app; do it only on authorised apps on your own test device.
- **Expecting Frida to just work on hardened apps.** Anti-instrumentation and Frida-detection block it; robust apps require bypassing those first (RE-protections skill).
- **Writing custom scripts for routine tasks.** Objection already does pinning/root bypass, storage dumps, and enumeration; use it for the common cases and script only what's custom.
- **Concluding a client-side control is "secure" because it's there.** Frida shows client-side controls (root detection, license checks, even pinning) can be bypassed by an attacker with device control; they shouldn't be the sole defence. That's a finding, not a Frida limitation.
- **Missing runtime-only data.** Static and storage analysis can't see decrypted values and in-memory keys; Frida is how you reach them — don't stop at static.

### References

- Frida documentation (frida.re) and Objection (github.com/sensepost/objection)
- OWASP MASTG (dynamic analysis, runtime instrumentation) and MASVS
- The ssl-pinning-bypass, reverse-engineering-protections, and insecure-data-storage skills
- Frida script repositories (codeshare.frida.re)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.