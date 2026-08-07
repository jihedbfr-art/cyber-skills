---
name: ios-static-analysis
domain: 05-mobile-security
description: Use when you have an iOS app (IPA) and want to read it statically for secrets, weak configuration, and binary protection gaps — without running it.
difficulty: intermediate
tags: [mobile, ios, static-analysis, ipa, reverse-engineering]
tools: [otool, class-dump, mobsf, hopper]
---

## Purpose

An iOS app ships as an IPA — an archive containing a compiled binary, resources, and configuration. Static analysis reads all of it without running the app: hardcoded secrets, insecure settings, exposed URLs, and whether the binary has the protections it should. iOS is more locked down than Android (encrypted binaries, stronger platform controls), which changes the analysis, but the goals are the same. This skill covers statically assessing an iOS app.

## When to use it

The opening pass on any iOS assessment, alongside the Android equivalent. It's static and safe; dynamic analysis and traffic inspection come after. Analyse only apps you're authorised to test.

## Procedure

1. **Unpack the IPA.** It's a zip — extract it to get the `.app` bundle (the binary, `Info.plist`, resources, embedded provisioning profile):
   ```
   unzip app.ipa            # -> Payload/AppName.app/
   ```
2. **Read `Info.plist` — the app's configuration.** It reveals a lot: URL schemes (deep-link surface), App Transport Security settings (`NSAppTransportSecurity` — is TLS enforced, or is `NSAllowsArbitraryLoads` set, disabling it?), permissions, and background modes.
3. **Extract strings and hunt for secrets.** Like Android, strings leak API keys, endpoints, and credentials:
   ```
   strings AppName            # then grep for keys, URLs, secrets
   ```
   Note: App Store binaries are encrypted (FairPlay); a binary pulled from the App Store may need decryption (from a jailbroken device) before strings are readable. A binary from the developer or a test build is usually unencrypted.
4. **Inspect the binary's protections** with `otool` — check for stack canaries, PIE/ASLR, and whether it's a release build:
   ```
   otool -hv AppName          # header, PIE flag
   otool -Iv AppName | grep stack_chk    # stack canaries
   ```
5. **Recover class/method structure** — `class-dump` (on an unencrypted binary) or a disassembler (Hopper/Ghidra) reveals the Objective-C/Swift class interfaces, exposing the app's logic and sensitive method names.
6. **Check data-storage and crypto usage** in the code/resources — insecure storage of secrets (see that skill), weak or hardcoded crypto, and disabled security features.
7. **Run an automated pass** (MobSF) for breadth and a checklist score, then verify findings by hand.

## Cheatsheet

```
IPA = zip (binary + Info.plist + resources). static = read without running.
  iOS more locked down (encrypted App Store binaries, stronger platform) — goals same as Android

1. UNPACK: unzip app.ipa -> Payload/App.app/
2. Info.plist (config, high value):
     URL schemes (deep-link surface) | NSAppTransportSecurity (NSAllowsArbitraryLoads = TLS disabled!)
     | permissions | background modes
3. STRINGS -> secrets/keys/URLs   (App Store binary = FairPlay-encrypted -> decrypt first from JB device;
     dev/test build usually unencrypted)
4. BINARY PROTECTIONS (otool): PIE/ASLR (otool -hv), stack canaries (otool -Iv | grep stack_chk)
5. CLASS STRUCTURE: class-dump (unencrypted) / Hopper / Ghidra -> ObjC/Swift interfaces + method names
6. data storage (insecure secrets) + crypto (weak/hardcoded/disabled features)
7. MobSF automated pass (breadth + score) -> verify by hand
```

## Reading the analysis

- **`NSAllowsArbitraryLoads: true` in Info.plist** = App Transport Security is disabled, so the app can make cleartext HTTP connections — a finding, since it undermines the TLS enforcement iOS gives by default. Check it explicitly.
- **A live secret in strings** = a direct leak the app ships to every device; the same as Android — anything in the binary is extractable. High value.
- **URL schemes in Info.plist** = the deep-link attack surface (ties into the deep-link skill); note them for how the app handles inbound links.
- **Missing binary protections** (no PIE, no stack canaries) = weaker exploitation resistance; a hardening gap, though release builds usually have them.
- **An encrypted App Store binary** = strings/class-dump won't work until decrypted (from a jailbroken device); a dev/test build is easier. Recognise which you have.
- **Class/method structure recovered** = the app's logic and sensitive methods exposed, guiding deeper analysis and the dynamic phase.
- **MobSF findings** = leads to verify, not verdicts; confirm each in the binary/resources.

## Pitfalls

- **Expecting to read an encrypted App Store binary directly.** FairPlay encryption means strings/class-dump fail until the binary is decrypted (from a jailbroken device). Use a dev/test build where possible, or decrypt first.
- **Skipping Info.plist.** It holds high-value config — ATS/TLS settings, URL schemes, permissions. The fastest findings are often here (like the Android manifest).
- **Missing `NSAllowsArbitraryLoads`.** It silently disables the platform's TLS enforcement; easy to overlook, important to flag.
- **Treating iOS like Android.** The binary format, encryption, and platform controls differ; the tools (otool, class-dump) and the encrypted-binary reality are iOS-specific.
- **Trusting the automated score.** MobSF flags possibilities; verify in the actual binary and resources.

## References

- OWASP Mobile Application Security Testing Guide (MASTG) — iOS static analysis
- OWASP MASVS; otool, class-dump, Hopper/Ghidra, MobSF documentation
- The android-static-analysis, insecure-data-storage, and deep-link-and-ipc-abuse skills
- Apple App Transport Security documentation
