---
format: "v2"
name: "dns-security"
title: "Dns Security"
title_fr: "Sécurité DNS"
description: "Use when securing DNS infrastructure — DNSSEC, resolver filtering, and detecting tunnelling and exfiltration — because DNS is both a control point and an attacker's favourite covert channel."
description_fr: "À utiliser pour sécuriser l'infrastructure DNS — DNSSEC, filtrage au niveau du résolveur et détection du tunneling et de l'exfiltration — car le DNS est à la fois un point de contrôle défensif et le canal furtif préféré des attaquants."
domain: "02-network-security"
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

DNS is the network's address book, and it's both a defensive control point and an attacker's favourite covert channel. Attackers spoof it, tunnel data and C2 through it (because DNS is almost always allowed out), and abuse it for exfiltration. Defenders can use it to block malicious domains and spot compromise. This skill covers securing DNS infrastructure and detecting the abuse that hides in DNS traffic.

### When to use it

Hardening DNS infrastructure, or investigating suspicious network activity where DNS is the channel. It complements the OSINT dns-recon skill (the attacker's view) with the defender's view — protecting integrity and catching abuse.

### The two sides

- **Protecting DNS integrity** — stopping spoofing/cache poisoning so users reach the real destinations (DNSSEC, resolver hygiene).
- **Using DNS as a control and detection point** — filtering malicious domains at the resolver, and detecting tunnelling/exfiltration in DNS traffic.

### Procedure

1. **Protect resolution integrity with DNSSEC** where appropriate — it signs DNS records so resolvers can verify they weren't tampered with, defeating cache poisoning and spoofing for signed zones. Sign your own zones and validate on your resolvers.
2. **Harden the resolvers.** Use trusted internal/upstream resolvers, restrict who can query them (don't run an open resolver — those get abused for amplification DDoS), and keep resolver software patched.
3. **Filter at the resolver — a strong, cheap control.** Point clients at a resolver that blocks known-malicious and newly-registered domains (DNS filtering / protective DNS). This stops much malware and phishing at the name-resolution step, before a connection is even made.
4. **Detect DNS tunnelling and exfiltration** — the covert-channel abuse. Watch for the signatures: an unusual volume of DNS queries from one host, abnormally long or high-entropy subdomain labels (data encoded in the query), lots of TXT/NULL record queries, and queries to a single domain at high frequency:
   ```
   # signs of tunnelling in DNS logs
   long random-looking subdomains: a3f9c1b2....exfil.attacker.com
   high query volume to one domain | many TXT/NULL queries | high entropy labels
   ```
5. **Monitor DNS logs** as a detection source generally — malware resolving C2 domains, DGA (algorithmically-generated) domains, and connections to known-bad names all show up in DNS before or instead of anywhere else.
6. **Log and retain** DNS query data so you can hunt and investigate (it's one of the highest-value logs for threat hunting — see that domain).

### Cheatsheet

```
protect integrity
  DNSSEC        sign your zones + validate on resolvers (anti-poisoning/spoofing)
  resolvers     trusted only, patched, NOT open (open resolver = DDoS amplifier)

control + detect
  filtering     resolver blocks malicious/newly-registered domains (protective DNS)
                -> stops malware/phishing at resolution, before connection
  tunnelling signs (detect in DNS logs)
    long/high-entropy subdomain labels   a3f9...c1b2.exfil.evil.com
    high query volume to ONE domain
    many TXT / NULL record queries
    DGA domains (algorithmic, random-looking)
  DNS logs = top-tier detection + hunting source (malware C2 resolves here)
```

### Reading the situation

- **Zones/resolvers without DNSSEC validation** = exposure to cache poisoning and spoofing (users silently sent to attacker destinations). Sign and validate where feasible.
- **An open resolver** = abused for amplification DDoS against others and a sign of poor hygiene; restrict it.
- **No resolver filtering** = a cheap, high-impact control left unused; malware and phishing resolve freely. Protective DNS closes much of this.
- **DNS tunnelling signatures** (long high-entropy labels, high volume to one domain, TXT/NULL floods) = active covert channel — likely C2 or exfiltration. A strong, often-missed detection.
- **DNS logs not collected** = losing one of the best detection and hunting sources; malware C2 resolution is invisible.
- **DNSSEC + filtered, patched resolvers + monitored DNS logs** = the strong posture.

### The fix / best practice

- **Deploy DNSSEC** for zones you control and enable validation on your resolvers to protect integrity.
- **Run hardened, non-open resolvers** — trusted upstreams, restricted querying, patched.
- **Adopt protective DNS / resolver filtering** to block malicious and newly-registered domains at resolution — one of the best effort-to-impact controls in this domain.
- **Detect tunnelling and abuse** — alert on the query-volume, label-entropy, and record-type signatures; block or investigate.
- **Collect, retain, and hunt DNS logs** — treat them as a primary detection source (feed into detection-engineering and threat-hunting).
- **Restrict outbound DNS** so clients can only use approved resolvers (stops malware using its own DNS to bypass filtering).

### Pitfalls

- **Ignoring DNS as an exfiltration channel.** DNS is almost always allowed outbound, which is exactly why attackers tunnel through it. If you're not watching DNS, you're blind to a common exfil/C2 path.
- **Running an open resolver.** It gets weaponised for amplification DDoS and signals weak hygiene. Restrict querying.
- **Skipping resolver filtering.** It's cheap and blocks a lot of malware/phishing at the earliest point; leaving it out wastes a strong control.
- **Not collecting DNS logs.** You lose a top detection and hunting source — malware's C2 resolution often shows here first.
- **Allowing clients to use arbitrary resolvers.** Malware then bypasses your filtering with its own DNS; force approved resolvers.

### References

- NIST SP 800-81 (Secure Domain Name System Deployment Guide)
- CISA Protective DNS guidance
- MITRE ATT&CK — T1071.004 (DNS), T1048 (exfiltration over alternative protocol)
- The threat-hunting (DNS hunting) and OSINT dns-recon skills

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.