---
format: "v2"
name: "mobile-forensics"
title: "Mobile Forensics"
title_fr: "Forensic mobile"
description: "Use when investigating a smartphone — the acquisition challenges, app data, and artefacts unique to iOS and Android, where the evidence is locked and encrypted differently than a PC."
description_fr: "À utiliser pour investiguer un smartphone — les défis d'acquisition, les données d'applications et les artefacts propres à iOS et Android, où la preuve est verrouillée et chiffrée différemment que sur un PC."
domain: "23-digital-forensics"
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

Phones hold as much evidence as any computer — messages, location, app activity, photos, communications — but getting to it is far harder. Modern phones are encrypted by default, locked, and controlled by the OS, so acquisition is the central challenge in a way it isn't for a PC. This skill covers the realities of mobile forensics: the acquisition levels, what evidence phones hold, and the iOS/Android specifics, so you know what's obtainable and how.

### When to use it

Investigations where a phone is evidence — many modern cases, since so much activity happens on mobile. It differs enough from computer forensics (acquisition-dominated, heavily tool-dependent, encryption-limited) that it's worth understanding as its own discipline.

### The acquisition challenge

Unlike a PC where you image the disk, phone acquisition is constrained by encryption, the lock, and OS restrictions — and comes in levels of completeness:

- **Manual** — scrolling through the phone and photographing what's visible. Last resort; limited and hard to defend.
- **Logical** — extracting data the device/backup API exposes (contacts, messages, call logs, some app data). Broadly available, but incomplete — you get what the API gives.
- **File system** — access to the filesystem, more app data and databases; needs more privileged access.
- **Physical** — a full bit-for-bit image; the most complete, but often impossible on modern encrypted devices without specialised tooling or exploits.

Modern encryption means physical/full access frequently isn't achievable without commercial tools (Cellebrite, GrayKey) or a supported exploit, and even then it's device/OS-version dependent.

### Procedure

1. **Preserve the device state — isolate it from networks.** A connected phone can be remotely wiped or altered; put it in a Faraday bag / airplane mode immediately to prevent remote commands and preserve evidence. Keep it charged (a dead phone may re-lock/re-encrypt).
2. **Determine what acquisition is possible** for this exact device and OS version — this dominates everything. Lock state (is it unlocked/known passcode?), OS version, and available tooling decide whether you get logical, file-system, or physical.
3. **Acquire at the highest level achievable**, using validated tools, and document the method (mobile evidence is heavily tool-mediated, so the tool and its version are part of the record).
4. **Analyse the extracted data — where the evidence is:**
   - **Communications:** SMS/iMessage, and third-party messengers (WhatsApp, Signal, etc.) stored in app databases (often SQLite).
   - **Location:** GPS in photos, app location history, cell/Wi-Fi records.
   - **App activity, browser history, photos/media, call logs, and account data.**
5. **Handle iOS vs Android differences** — iOS is more locked-down with strong encryption (acquisition often via backups or specialised tools); Android is fragmented (many vendors/versions), so techniques vary by device. Know which you're dealing with.
6. **Consider cloud backups** — much phone data syncs to iCloud/Google, which may be obtainable (with legal authority) and can fill gaps when the device itself is inaccessible.
7. **Maintain chain of custody and legal authority** — mobile evidence is highly personal and privacy-sensitive; ensure proper authorisation, and preserve/hash extractions as evidence.

### Cheatsheet

```
acquisition DOMINATES mobile forensics (encryption + lock + OS restrictions)

levels (least -> most complete)
  manual        photograph the screen (last resort)
  logical       API/backup data (contacts, msgs, calls) — common, incomplete
  file system   filesystem + app DBs (needs privileged access)
  physical      full bit-for-bit image — often impossible on modern encrypted devices
                without commercial tools (Cellebrite/GrayKey) or an exploit

FIRST: isolate from networks (Faraday/airplane) -> stop remote wipe; keep charged
then: determine what's POSSIBLE for this exact device+OS+lock state -> acquire highest
      document the tool + version (evidence is tool-mediated)

evidence: messages (SMS + app SQLite: WhatsApp/Signal...), location (GPS/history),
          app activity, browser, media, call logs, accounts
iOS = locked/encrypted (backups/specialised tools) | Android = fragmented (varies)
cloud backups (iCloud/Google) may fill gaps — with legal authority
chain of custody + authorisation (highly privacy-sensitive)
```

### Reading the situation

- **A locked, modern, encrypted device with no passcode** = acquisition may be limited to logical or blocked entirely; the achievable level dictates the whole investigation. This constraint is the defining reality of mobile forensics.
- **An unlocked device or known passcode** = far more is obtainable (file-system/physical via tools); the lock state is the single biggest factor.
- **Third-party messenger databases recovered** = the substantive evidence in many cases — chats, media, contacts in app SQLite stores.
- **Location artefacts** (photo GPS, app history) = often decisive for placing a person/device; a rich source unique to mobile.
- **A device that got network access before isolation** = risk of remote wipe/alteration; isolating first is why the evidence survived (or a warning if it didn't).
- **Data only in cloud backup** = when the device is inaccessible, iCloud/Google backups (with legal authority) may be the path — the account, not just the device.

### Pitfalls

- **Not isolating the device immediately.** A networked phone can be remotely wiped or altered, destroying evidence. Faraday bag / airplane mode first, always.
- **Assuming you can "just image it".** Modern encryption often makes full physical acquisition impossible without specialised tools/exploits; set expectations by what's actually achievable for that device and OS.
- **Letting it power down / re-lock.** A dead or rebooted phone may re-encrypt and become harder to access; keep it charged and in its current state.
- **Ignoring the iOS/Android divergence.** They differ fundamentally in acquisition; applying one's techniques to the other fails.
- **Overlooking cloud backups.** When the device won't yield, the synced account may — with proper legal authority.
- **Neglecting authorisation/privacy.** Phones are intensely personal; proper legal authority and chain of custody are non-negotiable.

### References

- NIST SP 800-101 (Guidelines on Mobile Device Forensics)
- Cellebrite / GrayKey / mobile-forensics tool documentation
- SANS mobile forensics (FOR585)
- The disk-imaging-and-hashing, chain-of-custody, and cloud-forensics skills

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.