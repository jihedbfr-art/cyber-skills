---
name: dns-and-proxy-hunting
domain: 20-threat-hunting
description: Use when hunting DNS and web-proxy logs for C2, exfiltration, and malicious domains — high-value logs that reveal what hosts are really talking to.
difficulty: intermediate
tags: [threat-hunting, dns, proxy, c2, exfiltration]
tools: []
---

## Purpose

DNS and web-proxy logs are among the richest hunting sources, because almost everything a host does on the internet passes through name resolution and web requests. Malware resolving C2 domains, data tunnelled out through DNS, connections to newly-registered or algorithmically-generated domains — it all shows here. This skill covers hunting these logs for the network signatures of compromise, a high-yield hunt available to almost any SOC.

## When to use it

Hunting for C2, exfiltration, and malicious-domain contact, especially when you don't have specific IoCs. DNS and proxy logs are widely collected and high-signal, making this one of the most accessible and productive network hunts.

## Procedure

1. **Get the DNS and proxy logs with enough history.** DNS query logs (what domains hosts resolve) and proxy logs (web requests, with URLs, user-agents, bytes, categories) are the data. History lets you spot patterns and rare/new domains.
2. **Hunt DNS tunnelling / exfiltration.** Data smuggled through DNS shows as anomalous query volume from a host, abnormally long or high-entropy subdomain labels (encoded data), lots of TXT/NULL queries, and high query frequency to one domain (ties into the DNS-security skill's signatures). One host resolving thousands of long random subdomains under one domain is exfiltration.
3. **Hunt newly-registered and algorithmic (DGA) domains.** Malware often uses freshly-registered domains (low age) or DGA (algorithmically-generated, random-looking) domains for C2. Hunt for connections to newly-seen/newly-registered domains and to domains with high randomness/entropy in the name — legitimate traffic rarely goes to these.
4. **Hunt suspicious proxy patterns.** Uncategorised or newly-categorised domains, unusual user-agents (a non-browser user-agent, or a tool's default), direct-to-IP requests, and abnormal request timing (beaconing — see that skill). The user-agent and category fields are especially useful.
5. **Hunt for known-bad and rare destinations.** Rare domains contacted by only one host, connections to uncategorised or high-risk categories, and (with threat intel) known-malicious domains. Rarity is a strong prioritisation signal — the domain nothing else talks to.
6. **Filter the enormous legitimate volume.** DNS/proxy logs are huge and mostly benign; baseline normal, allowlist the known-good (CDNs, major services), and focus on the anomalous. This filtering is the main effort.
7. **Investigate and operationalise** — a confirmed malicious domain becomes an IoC and a DNS/proxy block; the pattern becomes a detection (feeds detection, threat-intel, and DNS-security).

## Cheatsheet

```
DNS + proxy logs = richest hunting sources (almost all internet activity passes through)
  high-yield, widely collected. need HISTORY.

hunt these
  DNS TUNNELLING/exfil: high query volume | long/high-entropy subdomains | TXT/NULL floods
    | high frequency to one domain  (one host -> 1000s of random subdomains = exfil)
  NEW/DGA domains: low domain age (newly-registered) | high name randomness/entropy
    (legit traffic rarely goes here)
  PROXY anomalies: uncategorised/new-category domain | odd user-agent (non-browser/tool default)
    | direct-to-IP | beaconing timing
  RARE destinations: domain contacted by only ONE host | high-risk category | known-bad (intel)

FILTER huge legit volume: baseline + allowlist CDNs/major services -> anomalous surface
investigate -> IoC + DNS/proxy block + detection
```

## Reading the hunt

- **A host resolving thousands of long, high-entropy subdomains under one domain** = DNS tunnelling/exfiltration; the volume-and-entropy signature is unmistakable and DNS is a favourite exfil channel because it's usually allowed out. A top find.
- **Connections to newly-registered or DGA (random-looking) domains** = strong C2 indicators; legitimate traffic rarely goes to freshly-registered or algorithmic domains, so these stand out sharply.
- **A rare domain contacted by only one host** = high-value lead; rarity is one of the best prioritisation signals — the destination nothing else in the org talks to is worth investigating.
- **An unusual user-agent in proxy logs** (a tool's default, a non-browser string) = often malware or a script rather than a user; the user-agent field frequently betrays automated/malicious traffic.
- **Overwhelming legitimate volume** = the challenge; DNS/proxy logs are mostly benign (CDNs, major services), so baselining and allowlisting known-good is most of the work before the malicious surfaces.
- **A confirmed malicious domain** = block it at the resolver/proxy, extract as an IoC, and turn the pattern into a detection.

## Pitfalls

- **Not filtering the legitimate volume.** DNS/proxy logs are enormous and mostly benign; without baselining and allowlisting known-good, the malicious signal drowns. Filtering is the main effort.
- **Ignoring DNS as an exfil/C2 channel.** DNS is almost always allowed outbound, which is exactly why attackers use it; not hunting DNS leaves a common channel unwatched.
- **Hunting only known-bad domains.** Chasing IoCs misses C2 on new/DGA domains; hunt the behaviours (tunnelling signature, domain age/entropy, rarity) that catch unknown infrastructure.
- **Overlooking the user-agent and category fields.** Proxy metadata (user-agent, domain category, bytes) is high-signal for spotting non-human/malicious traffic; don't hunt on destination alone.
- **Insufficient history.** Spotting rare domains and repeating patterns needs a time span; a short window misses slow or infrequent activity.

## References

- The network dns-security, beaconing-detection, and detection log-source-coverage skills
- MITRE ATT&CK — T1071.004 (DNS), T1048 (Exfiltration Over Alternative Protocol), T1568 (Dynamic Resolution/DGA)
- The threat-intelligence domain (domain reputation) and operationalising-a-hunt skill
- Passive DNS and domain-age/reputation services
