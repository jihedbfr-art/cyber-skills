---
name: insecure-data-storage
domain: 05-mobile-security
description: Use when checking how a mobile app stores data — finding the secrets, tokens, and PII left in readable local storage that anyone with the device (or a backup) can extract.
difficulty: intermediate
tags: [mobile, data-storage, secrets, android, ios]
tools: [adb, sqlite, plutil]
---

## Purpose

A mobile app runs on a device you don't control, and anything it stores locally can be read by whoever has the device — especially a rooted/jailbroken one, or via a backup. The most common mobile finding is sensitive data (credentials, tokens, PII, keys) stored insecurely in local storage. This skill covers finding insecurely-stored data on Android and iOS, one of the highest-yield mobile assessments.

## When to use it

On any mobile assessment, after static analysis. It's high-yield because apps routinely store more than they should, unencrypted, in predictable places. Test only apps you're authorised to, on devices you control.

## Procedure

1. **Know where apps store data** and check each location:
   - **Android:** shared preferences (`/data/data/<pkg>/shared_prefs/*.xml`), SQLite databases (`/data/data/<pkg>/databases/`), internal/external files, and logs. External storage is world-readable — anything there is exposed.
   - **iOS:** `NSUserDefaults` (plist), Core Data / SQLite, files in the app sandbox, and the Keychain (the *secure* store).
2. **Exercise the app, then inspect storage.** Log in, use features, then examine what got written — credentials, session tokens, PII, or keys appearing in prefs/databases/files is the finding. On a rooted/jailbroken device you can read the app's private data directly:
   ```
   adb shell run-as <pkg> cat shared_prefs/*.xml      # Android (debuggable/rooted)
   # pull and inspect SQLite databases, plist files
   ```
3. **Check what should be in the secure store but isn't.** Secrets belong in the platform's secure storage — the **Android Keystore** and **iOS Keychain** — which are hardware-backed and protected. Secrets in plain preferences/plists/databases instead are the finding; the secure store exists precisely for this.
4. **Check logs.** Apps often log sensitive data (tokens, PII) to system logs, which other apps or anyone with the device can read:
   ```
   adb logcat            # watch for secrets logged during use
   ```
5. **Check backups.** Data included in device backups (Android `allowBackup`, iOS backups) can be extracted from the backup even without device access. Sensitive data that's backed up unencrypted is exposed via that channel.
6. **Assess caching and unintended storage** — cached API responses, keyboard cache, screenshots (the app snapshot iOS takes on backgrounding can capture sensitive screens), and WebView caches.
7. **Report by sensitivity** — a stored credential or token is high; a cached non-sensitive value is not. Judge by what's exposed.

## Cheatsheet

```
mobile app runs on a device you DON'T control -> local storage = readable by whoever has it
  (rooted/JB device, or a BACKUP). most common mobile finding.

where data lives
  Android: shared_prefs/*.xml | databases/ (SQLite) | files | logs | EXTERNAL storage (world-readable!)
  iOS: NSUserDefaults (plist) | Core Data/SQLite | sandbox files | KEYCHAIN (the secure store)

check
  exercise app (login, use) THEN inspect storage -> creds/tokens/PII/keys written = finding
    adb shell run-as <pkg> cat shared_prefs/*.xml ; pull SQLite/plist
  SECURE STORE: secrets belong in Android KEYSTORE / iOS KEYCHAIN (hardware-backed)
    -> secrets in plain prefs/plist/db = the finding
  LOGS: adb logcat -> sensitive data logged (readable by others)
  BACKUPS: allowBackup / iOS backup -> data extractable from backup w/o device access
  CACHING/unintended: cached API responses | keyboard cache | iOS backgrounding SCREENSHOT | WebView cache
report by SENSITIVITY (stored token = high ; cached non-sensitive = low)
```

## Reading the findings

- **Credentials, session tokens, or keys stored in plain preferences/plists/databases** = the classic high-value mobile finding; anyone with the device (or a backup) extracts them. These belong in the Keystore/Keychain, and their absence from it is the issue.
- **Secrets in the secure store (Keystore/Keychain)** = the correct pattern; hardware-backed and protected. Note it as done right.
- **Sensitive data on Android external storage** = world-readable by any app; a direct exposure. Nothing sensitive should be there.
- **Secrets in logs (`logcat`)** = readable by other apps/anyone with the device; a common and overlooked leak. Apps shouldn't log sensitive data.
- **Sensitive data in backups** = extractable from a backup without device access — a channel people forget; `allowBackup=true` on Android or unprotected iOS backup exposes it.
- **The iOS backgrounding screenshot capturing a sensitive screen** = a subtle leak (the snapshot is stored); apps should obscure sensitive screens on backgrounding.
- **Only non-sensitive cached data stored locally, secrets in the secure store** = the good state.

## The fix

- **Store secrets in the platform secure store** — Android Keystore, iOS Keychain (hardware-backed) — never in plain preferences, plists, databases, or files.
- **Don't store what you don't need.** Minimise sensitive data on the device; a token you don't persist can't be stolen from storage.
- **Never put sensitive data on Android external storage** (world-readable) or in logs.
- **Control backups** — exclude sensitive data from backups (`allowBackup=false` or backup exclusion rules; iOS data-protection classes).
- **Encrypt sensitive local data** at rest where it must be stored, using keys from the secure store.
- **Obscure sensitive screens on backgrounding** (iOS snapshot), and clear caches of sensitive data.

## Pitfalls

- **Storing secrets in plain preferences/plists.** The most common finding; the secure store (Keystore/Keychain) exists precisely for this. Use it.
- **Forgetting logs.** Sensitive data logged to `logcat`/system logs is readable by others; a frequent, overlooked leak. Don't log secrets.
- **Ignoring backups.** Data in backups is extractable without the device; sensitive data must be excluded or protected.
- **Sensitive data on external storage.** World-readable on Android; a direct exposure.
- **Missing the iOS backgrounding screenshot.** The OS snapshots the app on backgrounding; a sensitive screen gets captured and stored. Obscure it.
- **Only inspecting storage before using the app.** Sensitive data is written during use (login, activity); exercise the app first, then inspect.

## References

- OWASP MASTG (data storage testing) and MASVS (storage requirements)
- Android Keystore and iOS Keychain / Data Protection documentation
- The android-static-analysis, ios-static-analysis, and mobile-auth-and-biometrics skills
- adb, SQLite, plutil tooling
