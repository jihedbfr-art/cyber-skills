# 01 — OSINT & Reconnaissance

Before you touch a target you map it, and most of that map is public. This domain covers pulling an organisation's attack surface out of open sources: domains, subdomains, hosts, employees, leaked credentials, exposed services. Passive first, active only where the scope allows it.

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [subdomain-enumeration](01-subdomain-enumeration/SKILL.md) | Find the subdomains that widen the attack surface | ✅ |
| 02 | dns-recon | Zone data, records, and misconfig from DNS alone | TODO |
| 03 | google-dorking | Query operators that surface exposed files and panels | TODO |
| 04 | email-and-credential-leaks | Check breach corpora for org accounts | TODO |
| 05 | github-secret-recon | Find secrets and infra leaked in public repos | TODO |
| 06 | metadata-extraction | Pull names, paths, software from public documents | TODO |
| 07 | certificate-transparency | Discover hosts via CT logs | TODO |
| 08 | shodan-censys-recon | Fingerprint internet-facing services | TODO |
| 09 | social-media-profiling | Map staff and tech stack from public profiles | TODO |
| 10 | asn-and-ip-mapping | Tie IP ranges back to the target org | TODO |

Start with `subdomain-enumeration` — it feeds almost everything else in this domain and in web/API testing.
