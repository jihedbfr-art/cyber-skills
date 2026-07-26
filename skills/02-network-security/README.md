# 02 — Network Security

Everything that moves between hosts. Discovering what's listening, segmenting so a foothold doesn't become the whole network, and reading traffic when something looks wrong. Half of this domain is offensive (find the open port), half is architectural (why it shouldn't have been open).

| # | Skill | What it does | Status |
|---|-------|--------------|--------|
| 01 | [port-and-service-scanning](01-port-and-service-scanning/SKILL.md) | Map open ports and fingerprint services with nmap | ✅ |
| 02 | network-segmentation | Design and verify blast-radius boundaries | TODO |
| 03 | firewall-rule-review | Audit rule sets for shadowed and permissive rules | TODO |
| 04 | packet-capture-analysis | Read pcaps with tcpdump and Wireshark | TODO |
| 05 | tls-inspection | Check cipher suites, protocols, cert chains on the wire | TODO |
| 06 | vpn-security | Assess IPsec and WireGuard configs | TODO |
| 07 | dns-security | DNSSEC, filtering, tunnelling detection | TODO |
| 08 | mitm-and-arp-spoofing | Lab-only interception, and how to stop it | TODO |
| 09 | ids-ips-tuning | Cut false positives without going blind | TODO |
| 10 | network-access-control | 802.1X and NAC enforcement | TODO |

`port-and-service-scanning` is the entry point — you can't secure a service you didn't know was exposed.
