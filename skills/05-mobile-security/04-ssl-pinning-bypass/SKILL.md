---
format: "v2"
name: "ssl-pinning-bypass"
title: "Ssl Pinning Bypass"
title_fr: "Contournement du certificate pinning"
description: "Use in an authorised assessment to bypass certificate pinning so you can inspect a mobile app's TLS traffic — and understand what pinning does and doesn't protect."
description_fr: "À utiliser lors d'une évaluation autorisée pour contourner le certificate pinning et pouvoir inspecter le trafic TLS d'une application mobile — et pour comprendre ce que le pinning protège réellement, et ce qu'il ne protège pas."
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

Certificate pinning makes a mobile app trust only a specific certificate/key for its backend, so an interception proxy's certificate is rejected — which is good security, but blocks the legitimate traffic inspection an authorised assessor needs to test the app's API. This skill covers bypassing pinning *in an authorised test on a device you control* to inspect traffic, and understanding pinning's real security role. The goal is testing the app, not defeating a user's protection.

### When to use it

During an authorised mobile assessment, when you need to see the app's HTTPS traffic to test its API (the mobile-api-traffic skill) but pinning blocks your proxy. Only on apps you're authorised to test, on your own test device — bypassing pinning on someone else's app/device to intercept their traffic is an attack, not assessment.

### Understanding pinning first

- **What pinning does:** the app validates the server's certificate against a pinned value (a specific cert or public key), rejecting any other — even a valid CA-issued one. This defeats interception via a rogue/added CA certificate (the normal way a proxy intercepts TLS), protecting users against man-in-the-middle even if their device trust store is compromised.
- **What it doesn't do:** pinning protects the *channel*, not the endpoints. It doesn't stop an attacker who controls the device (root/jailbreak) from bypassing it, and it's not a substitute for backend security. It raises the bar for interception, which is exactly why an assessor has to bypass it to test.

### Procedure (authorised, own device)

1. **Set up the interception environment.** A test device (rooted/jailbroken makes bypass easier), an intercepting proxy (Burp/mitmproxy) with its CA certificate installed, and the device routed through the proxy. Without pinning, this alone would let you see traffic; pinning is what you're bypassing.
2. **Confirm pinning is the blocker.** If your proxy CA is installed but the app's traffic fails (connection errors, no traffic in the proxy) while other apps work, pinning is likely rejecting your certificate.
3. **Bypass with dynamic instrumentation.** The standard approach is hooking the app's TLS validation at runtime with Frida/Objection to disable the pinning check — this modifies the running app on your test device to accept your proxy's certificate:
   ```
   objection -g <package> explore     # then: android sslpinning disable / ios sslpinning disable
   # or a Frida universal-pinning-bypass script
   ```
4. **For static/other approaches**, you can also patch the app (modify the pinning logic and repackage) or, on Android, adjust the network security config in a repackaged app — but runtime hooking is usually fastest.
5. **Inspect the now-visible traffic** to test the API (the mobile-api-traffic skill) — the point of the bypass is to see and test what the app sends to its backend.
6. **Recognise the limits.** Some apps have robust, custom, or multiple pinning implementations that resist generic bypass; those need targeted analysis. Anti-tampering/root-detection may also fight you (the RE-protections skill).

### Cheatsheet

```
pinning: app trusts ONLY a specific cert/key -> rejects a proxy's cert -> blocks interception
  GOOD security (defeats MITM via rogue CA), but blocks authorised traffic testing

AUTHORISED test, OWN device only (bypass on someone else's app/device = attack)

what pinning does / doesn't
  DOES: protect the CHANNEL vs MITM even if device trust store compromised
  DOESN'T: stop an attacker who controls the device (root/JB) ; substitute for backend security

bypass (authorised, own test device)
  1. proxy (Burp/mitmproxy) + CA installed + device routed through it
  2. confirm pinning is the blocker (proxy CA installed but app traffic fails, others work)
  3. RUNTIME HOOK (standard): Frida/Objection disable pinning check
       objection -g <pkg> explore -> android/ios sslpinning disable
     (or patch app / adjust network-security-config + repackage)
  4. inspect traffic -> test the API (mobile-api-traffic)
limits: robust/custom/multiple pinning resists generic bypass ; anti-tamper/root-detection fights you
```

### Reading the situation

- **Proxy CA installed but the app's traffic fails while others work** = pinning is rejecting your certificate; that's the signal you need to bypass it to test. Confirm before assuming.
- **Objection/Frida pinning-disable working** = you can now see the traffic and test the API; the bypass succeeded on your test device. This is the assessment goal.
- **Generic bypass failing** = the app has robust, custom, or multiple pinning; it needs targeted analysis of the pinning implementation rather than an off-the-shelf script.
- **Anti-tampering/root-detection blocking the app on your rooted device** = the app is fighting instrumentation (the RE-protections skill); you may need to bypass those too to proceed.
- **Pinning present and robust** = from the *defensive* view, this is good — it raises the interception bar for real attackers. The point of the assessment is testing the app behind it, not concluding pinning is bad.

### The defensive view (why apps should pin)

Pinning is a *recommended* control, so the takeaway isn't "pinning is an obstacle" but "pinning is good, and here's what it does":
- **Implement pinning** to protect users against MITM even when their device's trust store is compromised (a rogue CA, a malicious proxy). It meaningfully raises the bar.
- **But don't rely on it alone** — a determined attacker with device control bypasses it, exactly as an assessor does. Pinning complements, doesn't replace, backend security and proper TLS.
- **Make it robust** (custom/multiple pinning, combined with anti-tampering) if the threat model warrants resisting bypass — though it can't be absolute against device-level attackers.

### Pitfalls

- **Bypassing pinning outside an authorised test.** Doing it on someone else's app/device to intercept their traffic is an attack. Authorised assessments, own test device only.
- **Concluding pinning is bad because it blocked you.** It's a recommended control that protects users; the assessment bypasses it to test the app, not to argue against it. Keep the defensive framing.
- **Expecting generic bypass to always work.** Robust/custom pinning resists off-the-shelf scripts; those need targeted analysis.
- **Forgetting anti-tampering.** Root/jailbreak detection and anti-instrumentation may block the app before you can bypass pinning; you may need to handle those first.
- **Treating pinning as sufficient defence.** It protects the channel, not the endpoints; it complements backend security and TLS, it doesn't replace them.

### References

- OWASP MASTG (network testing, pinning bypass) and MASVS
- Frida, Objection, and universal pinning-bypass scripts documentation
- The mobile-api-traffic and reverse-engineering-protections skills
- OWASP Certificate and Public Key Pinning guidance (the defensive side)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.