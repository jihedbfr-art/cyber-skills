---
format: "v2"
name: "port-and-service-scanning"
title: "Port And Service Scanning"
title_fr: "Analyse des ports et des services"
description: "Use when you need to know what's actually listening on a host or range — open ports, the services behind them, and their versions — before assessing or hardening it."
description_fr: "À utiliser pour établir un inventaire fiable de ce qui écoute réellement sur un hôte ou une plage d'adresses — ports ouverts, services associés et leurs versions — avant toute évaluation ou tout durcissement."
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

You can't defend or attack a service you don't know is running. This skill takes you from an IP or range to a reliable inventory of open ports, the service on each, and its version — the map everything else in network and host security builds on.

### When to use it

Any time you're handed a host, a range, or a fresh cloud subnet and need ground truth about its exposure. On the defensive side, it's how you verify that a firewall change actually closed what you think it closed.

For internet-facing scans, make sure the target is yours or explicitly in scope. Scanning ranges you don't own draws abuse complaints and, in some places, charges.

### Procedure

1. Confirm scope and note the IPs/ranges in writing. If you're scanning across the internet, expect that the source IP will be logged.
2. Find live hosts first so you don't waste time port-scanning dead space:
   ```
   nmap -sn 10.0.0.0/24 -oA hosts-up
   ```
3. Do a fast wide sweep to learn which ports are open at all, then scan deeply only those. On large ranges, a fast scanner up front saves hours:
   ```
   masscan 10.0.0.5 -p1-65535 --rate 1000 -oL ports.txt
   ```
4. Run nmap's version and default-script scan against just the open ports for detail:
   ```
   nmap -sV -sC -p 22,80,443,3306 10.0.0.5 -oA detailed
   ```
5. For UDP, scan a focused set — full UDP is slow and noisy, so target the services that matter (DNS, SNMP, NTP, IKE):
   ```
   nmap -sU --top-ports 20 10.0.0.5 -oA udp
   ```
6. Save all three output formats (`-oA`) so you have grepable and XML versions for later tooling.

### Cheatsheet

```bash
nmap -sV --top-ports 1000 target -oA quick

nmap -sS -sV -sC -p- -T4 target -oA full

nmap -p- --open -T4 target

nmap -sV -p 80,443,8080,8443 target

naabu -host target -silent | nmap -sV -iL - -oA combined
```

Common `-T` timing: `-T4` for most engagements, `-T2` when you need to stay quiet or avoid tripping rate limits, `-T3` (default) when unsure.

### Reading the output

- **Open vs filtered.** `open` answered. `filtered` means a firewall ate the probe — no answer either way. `closed` answered with a reset. Filtered ports are a defensive signal, not necessarily a dead end.
- **Version strings are leads.** `Apache httpd 2.4.49` isn't just informational — that version has a path-traversal-to-RCE CVE. Match versions against known vulnerabilities before you get excited, and confirm; banners lie or get spoofed.
- **Unexpected ports.** A database port (3306, 5432, 27017) reachable from outside its tier is a finding on its own, regardless of version.
- **Default-script hits.** `-sC` output often includes anonymous FTP, exposed SMB shares, or weak TLS — read it, don't skim past it.

### Pitfalls

- **Scanning too fast on fragile networks.** `-T5` or a high masscan rate can knock over old devices, VoIP gear, or SCADA. Slow down on anything you don't recognise.
- **Trusting the banner.** Version detection is a guess based on responses. Confirm before reporting a CVE as present.
- **Forgetting UDP entirely.** Plenty of real exposure (SNMP with `public`, open DNS resolvers) is UDP-only and invisible to a TCP scan.
- **One scan, one moment.** Hosts come and go. A clean scan today isn't a clean scan next week — for monitoring, schedule it.

### References

- Nmap Reference Guide (nmap.org/book)
- Masscan README (github.com/robertdavidgraham/masscan)
- OWASP WSTG — Fingerprint Web Server, Enumerate Infrastructure

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.