# 02 — Network Security

Everything that moves between hosts. Discovering what's listening, segmenting so a foothold doesn't become the whole network, and reading traffic when something looks wrong. Half of this domain is offensive (find the open port), half is architectural (why it shouldn't have been open).

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [port-and-service-scanning](01-port-and-service-scanning/SKILL.md) | Map open ports and fingerprint services with nmap | ✅ |
| 02 | [network-segmentation](02-network-segmentation/SKILL.md) | Design and verify blast-radius boundaries | ✅ |
| 03 | [firewall-rule-review](03-firewall-rule-review/SKILL.md) | Audit rule sets for shadowed and permissive rules | ✅ |
| 04 | [packet-capture-analysis](04-packet-capture-analysis/SKILL.md) | Read pcaps with tcpdump and Wireshark | ✅ |
| 05 | [tls-inspection](05-tls-inspection/SKILL.md) | Check cipher suites, protocols, cert chains on the wire | ✅ |
| 06 | [vpn-security](06-vpn-security/SKILL.md) | Assess IPsec and WireGuard configs | ✅ |
| 07 | [dns-security](07-dns-security/SKILL.md) | DNSSEC, filtering, tunnelling detection | ✅ |
| 08 | [mitm-and-arp-spoofing](08-mitm-and-arp-spoofing/SKILL.md) | Lab-only interception, and how to stop it | ✅ |
| 09 | [ids-ips-tuning](09-ids-ips-tuning/SKILL.md) | Cut false positives without going blind | ✅ |
| 10 | [network-access-control](10-network-access-control/SKILL.md) | 802.1X and NAC enforcement | ✅ |

This domain is complete (10/10). `port-and-service-scanning` is the entry point — you can't secure a service you didn't know was exposed.

---
<p align="center"><sub><b>JihedAiLabs</b> &middot; part of the <a href="../../README.md">cyber-skills</a> library</sub></p>