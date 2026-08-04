---
name: shodan-censys-recon
domain: 01-osint-and-reconnaissance
description: Use when fingerprinting an organisation's internet-facing services through search engines that scan the whole internet — finding exposed services without sending them a packet yourself.
difficulty: beginner
tags: [osint, shodan, censys, recon, passive, attack-surface]
tools: [shodan, censys]
---

## Purpose

Shodan and Censys continuously scan the entire internet and index what's listening — services, banners, versions, certificates, exposed devices. Querying them tells you an organisation's internet-facing attack surface without you sending a single packet to the target; *they* did the scanning. This skill covers using these engines to map exposure, and it's about as passive as active-service recon gets.

## When to use it

External recon to see what an organisation exposes to the internet — open services, outdated software, exposed databases, IoT/industrial devices, accidentally-public dashboards. Also a strong self-audit: "what does the internet already know is listening on our IPs?" It complements port scanning (the network domain) by giving you a picture without touching the target.

## Procedure

1. **Search by the org's IP ranges and domains.** Once you know the target's IP space (from the ASN/IP-mapping skill) and hostnames, query for what's exposed:
   ```
   # Shodan
   shodan search 'org:"Example Corp"'
   shodan search 'net:203.0.113.0/24'
   shodan search 'hostname:example.com'
   ```
2. **Identify exposed services and versions** — the banners reveal what's running and often the version, feeding vulnerability targeting. Look for services that shouldn't face the internet.
3. **Hunt the high-risk exposures** specifically — databases, remote-access services, and admin interfaces open to the world:
   ```
   shodan search 'net:<range> port:3389'          # RDP exposed
   shodan search 'net:<range> product:MongoDB'     # database exposed
   ```
4. **Check for known-vulnerable and default-configured devices** — Shodan flags many CVEs and exposed IoT/industrial systems. An outdated service or a device with default credentials is a direct lead.
5. **Read certificates and metadata** — Censys is especially strong on certificate and service detail, which can tie hosts back to the org and reveal more names.
6. **Corroborate before reporting** — these engines cache; a result may be stale. Confirm the exposure is current (carefully, and only within scope) before treating it as live.

## Cheatsheet

```
Shodan queries
  org:"Example Corp"              by organisation
  net:203.0.113.0/24             by IP range
  hostname:example.com           by hostname
  ssl:"Example Corp"             by certificate org
  net:<range> port:3389          specific exposed service (RDP)
  net:<range> product:MongoDB     specific product

high-risk exposures to hunt
  databases (MongoDB, Elastic, Redis, MySQL) open to internet
  RDP (3389) / VNC / SSH exposed
  admin panels / dashboards (Kibana, Grafana, Jenkins) public
  outdated service versions with known CVEs
  IoT / ICS / cameras with default configs

Censys: strong on certificates + service detail (censys.io/search)
note: results are from THEIR scans (passive for you) but may be STALE — verify
```

## Reading the output

- **A database exposed to the internet** (MongoDB, Elasticsearch, Redis — historically often unauthenticated) = critical; direct data exposure without even an exploit. Among the highest-value finds.
- **RDP/VNC/SSH open to the world** = a brute-force and exploitation target; note it for the exposure it represents.
- **A public admin dashboard** (Jenkins, Grafana, Kibana) = often unauthenticated or weakly protected, and a foothold. High value.
- **Outdated service versions** flagged with CVEs = targeting leads; confirm the version and vulnerability before relying on it.
- **Devices tied to the org via certificate/banner** = expands the confirmed attack surface, sometimes revealing hosts you didn't know were theirs.
- **A stale result** = the engine's last scan may predate a change; verify current state within scope before acting.

## The fix / defensive use

- **Self-monitor** — query Shodan/Censys for your own IP ranges regularly to see your exposure exactly as attackers do; set up monitoring alerts for new exposed services.
- **Close what shouldn't be exposed** — databases and admin interfaces belong behind the network, not on the internet (ties into the security-group-review and network-segmentation skills). Move them off public IPs.
- **Patch or remove outdated exposed services** flagged with CVEs.
- **Fix default configurations** on any exposed device (credentials, unauthenticated access).
- **Reduce the banner information** where practical, though the real fix is not exposing the service in the first place — obscuring the banner doesn't help if the service shouldn't be reachable.

## Pitfalls

- **Assuming results are live.** These are cached scan results; a service may have been closed or changed since the last scan. Verify before treating as current.
- **Only searching by domain.** Much exposure is on IPs with no matching hostname; search by the org's IP ranges (from ASN mapping) too, or you'll miss it.
- **Treating a banner version as ground truth.** Banners can be wrong or spoofed; confirm before reporting a specific CVE as present.
- **Forgetting the self-audit angle.** The same query an attacker runs against you is your best exposure check — use it defensively and continuously.

## References

- Shodan (shodan.io) and Censys (censys.io) documentation and query syntax
- OWASP WSTG-INFO — fingerprinting and infrastructure enumeration
- The ASN-and-IP-mapping skill (to get the ranges to search) and network port-scanning skill
