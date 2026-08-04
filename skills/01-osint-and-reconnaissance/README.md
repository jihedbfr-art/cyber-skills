# 01 — OSINT & Reconnaissance

Before you touch a target you map it, and most of that map is public. This domain covers pulling an organisation's attack surface out of open sources: domains, subdomains, hosts, employees, leaked credentials, exposed services. Passive first, active only where the scope allows it.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [subdomain-enumeration](01-subdomain-enumeration/SKILL.md) | Find the subdomains that widen the attack surface | ✅ |
| 02 | [dns-recon](02-dns-recon/SKILL.md) | Zone data, records, and misconfig from DNS alone | ✅ |
| 03 | [google-dorking](03-google-dorking/SKILL.md) | Query operators that surface exposed files and panels | ✅ |
| 04 | [email-and-credential-leaks](04-email-and-credential-leaks/SKILL.md) | Check breach corpora for org accounts | ✅ |
| 05 | [github-secret-recon](05-github-secret-recon/SKILL.md) | Find secrets and infra leaked in public repos | ✅ |
| 06 | [metadata-extraction](06-metadata-extraction/SKILL.md) | Pull names, paths, software from public documents | ✅ |
| 07 | [certificate-transparency](07-certificate-transparency/SKILL.md) | Discover hosts via CT logs | ✅ |
| 08 | [shodan-censys-recon](08-shodan-censys-recon/SKILL.md) | Fingerprint internet-facing services | ✅ |
| 09 | [social-media-profiling](09-social-media-profiling/SKILL.md) | Map staff and tech stack from public profiles | ✅ |
| 10 | [asn-and-ip-mapping](10-asn-and-ip-mapping/SKILL.md) | Tie IP ranges back to the target org | ✅ |

This domain is complete (10/10). Start with `subdomain-enumeration` — it feeds almost everything else in this domain and in web/API testing.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>