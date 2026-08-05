---
name: browser-and-app-forensics
domain: 23-digital-forensics
description: Use when reconstructing user activity from browser and application data — history, downloads, cached data, and app databases that reveal what a user did and when.
difficulty: intermediate
tags: [forensics, browser, applications, user-activity, sqlite]
tools: [hindsight, sqlite]
---

## Purpose

A huge amount of user activity lives in browser and application data — sites visited, files downloaded, searches typed, messages, and cached content. When an investigation needs to establish what a *user* did (insider cases, phishing victims, policy violations, or an attacker using a browser), this is where the evidence is. This skill covers recovering activity from browsers and application stores, much of which is SQLite databases waiting to be read.

## When to use it

Investigations centred on user behaviour — how a phishing victim reached a malicious site, what an insider accessed or exfiltrated, what an attacker did through a browser session. It complements host-artefact analysis with the user-activity layer.

## Procedure

1. **Locate the browser profile data.** Chrome/Edge/Firefox store history, downloads, cookies, and cache in per-user profile directories — mostly **SQLite databases** (`History`, `Cookies`, `Login Data`, etc.) plus cache files. Collect these from the image.
2. **Parse browser history and downloads — the core.** History shows URLs visited with timestamps and visit counts; downloads show what was fetched and from where. A tool like Hindsight parses Chromium browsers comprehensively; otherwise query the SQLite directly:
   ```
   # Hindsight: parses Chrome/Edge history, downloads, cookies, cache -> timeline
   # or read the SQLite directly:
   sqlite3 History "SELECT url, title, last_visit_time FROM urls ORDER BY last_visit_time;"
   ```
3. **Recover searches and typed data** — search terms and typed URLs reveal intent (what the user was looking for), often more telling than the pages themselves.
4. **Examine cache and cookies** — cache holds copies of viewed content (even from now-offline pages); cookies show authenticated sessions and site access. Cache can reconstruct what a page looked like when viewed.
5. **Check application data** — messaging apps, email clients, cloud-storage clients, and many desktop apps store data in SQLite or similar; the same reading techniques apply. These reveal communications and file activity.
6. **Account for private browsing and sync** — incognito/private mode leaves less on disk (but may still leave traces in memory or DNS), and browser sync means activity may exist across the user's other devices/account.
7. **Build into the timeline** and handle as evidence (work from the image, preserve, document) — browser timestamps feed the super-timeline for correlation with system events.

## Cheatsheet

```
where: per-user browser profile dirs — mostly SQLite databases + cache
  Chrome/Edge: History, Downloads, Cookies, Login Data, Web Data (SQLite)
  Firefox: places.sqlite (history+bookmarks), cookies.sqlite

parse (Chromium)
  Hindsight   history+downloads+cookies+cache -> unified timeline (the go-to)
  sqlite3 History "SELECT url,title,last_visit_time FROM urls ..."

what to recover
  history + downloads   sites visited (times/counts), files fetched + source
  searches / typed URLs intent (what they looked for) — often most telling
  cache                 copies of viewed content (even now-offline pages)
  cookies               authenticated sessions, site access

apps: messaging/email/cloud clients often SQLite too -> same techniques

caveats: private mode = less on disk (check memory/DNS) ; SYNC = data on other devices
feed the super-timeline ; work from the image; preserve
```

## Reading the data

- **Browser history + downloads around the incident** = how a user reached a malicious site and what they downloaded (the dropper/payload source) — central for phishing-victim and drive-by cases.
- **Search terms / typed URLs** = user intent, which pages alone don't show; "how to exfiltrate data" or a deliberately-typed malicious URL is strong evidence of intent.
- **Cached content** = what a page actually contained when viewed, recoverable even if the site is gone — useful when the live page can't be checked.
- **Cookies/session data** = which authenticated sites were accessed; relevant for account misuse and session-based attacks.
- **App SQLite databases** (messaging, cloud clients) = communications and file transfers — often the evidence in insider and data-exfiltration cases.
- **Sparse on-disk history where you expected activity** = possible private browsing or clearing; check memory, DNS, and synced devices rather than concluding nothing happened.

## Pitfalls

- **Forgetting most of it is SQLite.** Browser and app data are readable databases; you don't need special magic, just to know where they are and to parse timestamps correctly (browser time formats vary — WebKit/Chrome epoch differs from Unix).
- **Ignoring sync.** Cleared local history doesn't mean the activity is gone — it may be synced to the account and recoverable from other devices. Consider the whole account.
- **Assuming private mode leaves nothing.** It leaves less on disk but traces can remain in memory, DNS cache, and other artefacts. Absence on disk isn't proof.
- **Mishandling timestamps.** Browsers use different epoch formats; a mis-converted time corrupts the timeline. Use tools that handle it or convert carefully.
- **Working on the live profile.** Opening the browser or the live profile changes the data; parse from the forensic image.

## References

- Hindsight (Chromium browser forensics) and browser SQLite schema references
- SANS browser forensics resources
- The windows-artefacts, timeline-analysis, and chain-of-custody skills
- SQLite documentation (for direct database queries)
