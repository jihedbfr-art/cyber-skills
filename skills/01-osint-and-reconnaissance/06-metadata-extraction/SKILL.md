---
name: metadata-extraction
domain: 01-osint-and-reconnaissance
description: Use when mining an organisation's public documents for hidden metadata — usernames, software versions, internal paths, and names that leak from published files.
difficulty: beginner
tags: [osint, metadata, documents, recon, information-disclosure]
tools: [exiftool, metagoofil, foca]
---

## Purpose

Every document an organisation publishes — PDFs, Office files, images — carries metadata the authors never meant to share: the username who created it, the software and version, internal file paths, printer names, sometimes GPS coordinates on photos. Harvest a company's public documents and their metadata paints a picture of internal usernames, software in use, and naming conventions. This skill covers extracting and using that metadata. It's passive: you analyse published files, you don't touch the target's systems.

## When to use it

External recon to build a picture of an organisation's internal environment (usernames for password attacks, software versions for vulnerability targeting), or a self-audit of what your published documents leak. Pairs with subdomain enumeration and Google dorking (which find the documents).

## Procedure

1. **Collect the org's public documents.** Search engines and dorking surface them; automated tools can crawl a domain for downloadable files. Gather PDFs, DOCX, XLSX, PPTX, and images:
   ```
   metagoofil -d example.com -t pdf,docx,xlsx -o loot     # crawl + download public docs
   ```
2. **Extract the metadata** from each file. ExifTool reads metadata from virtually any file type:
   ```
   exiftool -a -u loot/*.pdf loot/*.docx
   ```
3. **Mine for usernames** — the `Author`, `Creator`, and `Last Modified By` fields reveal account names, exposing the org's username convention (e.g. `jsmith`, `john.smith`) for password spraying and phishing.
4. **Mine for software and versions** — the `Producer`/`Application` fields show which software (and version) created the file, hinting at what's in use internally and what might be outdated/vulnerable.
5. **Mine for internal paths and infrastructure** — embedded file paths (`C:\Users\jsmith\...`, network share names), printer names, and template sources reveal internal structure.
6. **Check images for GPS/EXIF** — photos may carry location and device data.
7. **Aggregate.** One document is a data point; a corpus reveals the username pattern, the software stack, and internal naming — that aggregate is the real value.

## Cheatsheet

```bash
# collect public documents from a domain
metagoofil -d example.com -t pdf,doc,docx,xls,xlsx,ppt,pptx -o loot
# (FOCA is a GUI alternative that does collect + analyse)

# extract metadata (all fields, including unknown/duplicate)
exiftool -a -u -g1 file.pdf
exiftool loot/*                        # batch

# the high-value fields
Author / Creator / Last Modified By   -> usernames + naming convention
Producer / Application / Software      -> software + versions in use
file paths in metadata                 -> internal structure (C:\Users\..., shares)
GPS / EXIF (images)                    -> location, device
```

## Reading the output

- **Usernames in Author/Creator fields** = the org's account-naming convention (`first.last`, `flast`, etc.) — directly useful for password spraying and targeted phishing. Often the highest-value find.
- **Software and versions** = the internal stack; an outdated version named in metadata is a targeting lead for known vulnerabilities.
- **Internal file paths / share names** = a map of internal structure and sometimes usernames again (`C:\Users\jsmith\`).
- **A consistent pattern across many documents** = confidence that the username/software convention is real, not a one-off — the aggregate is what matters.
- **GPS in images** = physical location exposure, relevant for people-focused recon or physical assessments.

## The fix (for your own documents)

- **Strip metadata before publishing.** Scrub documents on the way out — Office has a "Document Inspector"/remove-personal-info feature, and tools like ExifTool can bulk-clean files. Make it part of the publishing process.
- **Set generic authorship** where possible (a role/department rather than a personal username) so published files don't leak account names.
- **Audit already-published documents** — the same extraction you'd run as an attacker tells you what's already exposed; clean or re-publish the worst offenders.
- **Policy and awareness** — authors rarely realise documents carry this; a simple "scrub before publishing" step closes most of it.
- **Understand the username exposure feeds other attacks** — once the convention is public, pair the fix with MFA and anti-spraying controls, since you can't recall what's already out.

## Pitfalls

- **Dismissing it as low-value.** Individually a document leaks little; the *corpus* reveals your username convention and software stack — exactly what an attacker needs to target people and systems. Judge it in aggregate.
- **Cleaning new documents but ignoring the archive.** Years of published files may already be indexed; audit and remediate what's out there.
- **Assuming "delete personal info" catches everything.** Different file types hide metadata in different places; verify with ExifTool that it's actually gone.
- **Overlooking images.** People focus on documents and forget photos carry EXIF/GPS.

## References

- ExifTool documentation (exiftool.org)
- OWASP WSTG-INFO — metadata and information leakage
- Metagoofil / FOCA documentation
- CWE-200 (Exposure of Sensitive Information)
