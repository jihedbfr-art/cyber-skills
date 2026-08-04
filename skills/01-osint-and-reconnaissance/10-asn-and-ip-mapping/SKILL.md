---
name: asn-and-ip-mapping
domain: 01-osint-and-reconnaissance
description: Use when tying an organisation's IP ranges back to it — mapping ASNs, netblocks, and cloud allocations so you know the full IP footprint that belongs to the target.
difficulty: beginner
tags: [osint, asn, ip-ranges, recon, attack-surface]
tools: [whois, bgp-tools, amass]
---

## Purpose

Subdomains and certificates give you hostnames; this skill gives you the IP space behind them. Organisations own or are allocated ranges of IP addresses, often grouped under an Autonomous System Number (ASN). Mapping the target's ASNs and netblocks tells you the full range of IPs that belong to it — which then feeds Shodan searches, port scanning, and a complete picture of the attack surface, including hosts that have no hostname at all.

## When to use it

Early recon, to establish the IP scope before service-level enumeration. It's what turns "these few hostnames resolve to these IPs" into "here is the organisation's entire IP footprint". Also essential for defining scope precisely — you want to know which ranges genuinely belong to the target (and are in scope) before scanning anything.

## Procedure

1. **Start from known IPs.** Resolve the target's hostnames (from subdomain enumeration) to get seed IP addresses.
2. **Look up who owns each IP** with WHOIS — it returns the netblock, the owning organisation, and the ASN:
   ```
   whois 203.0.113.10          # -> netblock, org name, ASN
   ```
3. **Find the organisation's ASN(s)** and enumerate all the netblocks announced under them. An ASN groups the ranges an organisation routes, so this expands a single IP into the org's full announced space:
   ```
   # via BGP tools / bgp.he.net / whois -h whois.radb.net
   whois -h whois.radb.net -- '-i origin ASxxxxx'      # prefixes for an ASN
   ```
4. **Distinguish owned vs cloud-hosted ranges.** Many orgs host in AWS/Azure/GCP, so their hosts sit in the *cloud provider's* IP space, not their own ASN. Those IPs won't map back to the org via ASN — you tie them to the target through hostnames, certificates, and reverse DNS instead. Note which ranges are the org's own vs cloud (this also affects scope and authorisation).
5. **Reverse-DNS the ranges** to find hostnames tied to the org and confirm which IPs are really theirs:
   ```
   # reverse lookups across a netblock to surface org-related PTR records
   ```
6. **Build the confirmed IP scope** — the netblocks that genuinely belong to the target — and feed it into Shodan/Censys recon and (where authorised) port scanning.

## Cheatsheet

```bash
# who owns an IP? -> netblock + org + ASN
whois 203.0.113.10

# all prefixes announced by an ASN (the org's own space)
whois -h whois.radb.net -- '-i origin AS15169'
# or use bgp.he.net / bgp.tools (web) to browse an org's ASN + prefixes

# tie ranges back to the org
reverse DNS (PTR) across the netblock
certificates (Censys) + hostnames resolving into the range

# owned vs cloud
org's own ASN        -> their netblocks
AWS/Azure/GCP ranges -> tie via hostname/cert/PTR, NOT the org's ASN
                        (and mind scope/authorisation on shared cloud IPs)

# feeds: Shodan (net:<range>), port scanning, full attack-surface picture
```

## Reading the output

- **The organisation's own ASN and netblocks** = the core owned IP footprint; every IP in these ranges is a candidate host, including ones with no hostname that subdomain enumeration would never find.
- **Hosts in cloud-provider ranges** = the org runs in the cloud; you can't claim those IPs via ASN, and scanning shared cloud space raises scope/authorisation concerns — tie individual hosts to the org via hostname/cert/PTR instead.
- **Reverse-DNS records referencing the org** = confirmation that a range (or specific IPs) belong to the target.
- **A large owned IP space** = a bigger attack surface than the hostname list suggested — many IPs may host services with no DNS name, which is exactly what this skill surfaces.
- **Ambiguous ownership** (shared hosting, a range used by multiple orgs) = be cautious about scope; not every IP near the target's is the target's.

## The fix / defensive use

This is primarily a scoping and awareness skill rather than a vulnerability, but it has defensive value:

- **Know your own IP footprint.** Many organisations can't cleanly enumerate every range and cloud allocation they own — and unknown IP space is unmonitored attack surface. Map your ASNs and cloud ranges so nothing is unaccounted for.
- **Monitor your ranges** for exposed services (feed them into the Shodan/CSPM self-audit) — you can only watch the IP space you know is yours.
- **Get scope right** — for defenders authorising a test, providing the accurate owned ranges (and clarifying cloud-hosted assets that need provider authorisation) prevents both missed coverage and out-of-scope scanning.
- **Decommission forgotten allocations** — old netblocks and cloud ranges that still route to something are exactly where neglected, vulnerable hosts live.

## Pitfalls

- **Assuming every nearby IP belongs to the target.** Netblocks are shared, especially in hosting and cloud; verify ownership (WHOIS, PTR, certs) before treating an IP as in scope.
- **Missing cloud-hosted assets.** ASN mapping finds owned ranges but not the org's hosts sitting in AWS/Azure/GCP space — you need hostname/cert correlation for those, and they carry their own authorisation rules.
- **Scanning out of scope.** Expanding IP scope is powerful and risky — scanning ranges you're not authorised to test (or shared cloud IPs) crosses lines. Confirm ownership and authorisation first.
- **Treating it as complete on its own.** ASN mapping is one input; combine with subdomain, CT, and Shodan recon for the full picture — each finds hosts the others miss.

## References

- WHOIS and RADB (whois.radb.net) for ASN/prefix lookups
- bgp.he.net / bgp.tools for browsing ASNs and announced prefixes
- OWASP WSTG-INFO — enumerate infrastructure
- The Shodan/Censys and subdomain-enumeration skills (which consume the ranges this produces)
