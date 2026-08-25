---
format: "v2"
name: "mitm-and-arp-spoofing"
title: "Mitm And Arp Spoofing"
title_fr: "MITM et usurpation ARP"
description: "Use in a lab or authorised test to understand adversary-in-the-middle attacks on a local network — ARP spoofing and interception — and, crucially, the controls that stop them."
description_fr: "À utiliser en laboratoire ou dans le cadre d'un test autorisé pour comprendre les attaques de l'adversaire au milieu sur un réseau local — usurpation ARP et interception — et, surtout, les contrôles qui les arrêtent."
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

On a local network, an attacker who can position themselves between two hosts sees and can alter their traffic — an adversary-in-the-middle. ARP spoofing is the classic technique: poison the ARP tables so traffic meant for the gateway flows through the attacker first. This skill covers understanding the attack (in a lab or authorised test) so you can recognise and defend against it — the defensive payoff is the point, since the attack itself is only legal on networks you own or are authorised to test.

### When to use it

Learning how local-network interception works in a lab, an authorised internal assessment demonstrating the risk of a flat/unprotected LAN, or understanding what an attacker on your network segment can do so you can defend it. Never on a network you don't own or aren't authorised to test — intercepting others' traffic is illegal.

### How it works (the attack, for defenders)

ARP has no authentication — a host believes whatever ARP replies it receives. The attacker sends forged ARP replies telling the victim "the gateway's MAC is mine" and telling the gateway "the victim's MAC is mine". Both now send their traffic to the attacker, who forwards it on (staying invisible) while reading or modifying it. From that position the attacker can capture credentials in cleartext protocols, attempt SSL-strip on unprotected HTTP, and manipulate traffic.

### Procedure (lab / authorised only)

1. **Confirm you're on an authorised network** (your lab, or in-scope engagement). This attack affects other users' traffic, so scope discipline is absolute.
2. **Position between victim and gateway** with ARP poisoning, enabling forwarding so the victim stays connected (a silent MITM, not a DoS):
   ```
   bettercap -iface eth0            # then: set arp.spoof on, targets
   # or classic:
   arpspoof -i eth0 -t <victim> <gateway>
   ```
3. **Observe what's exposed** — capture the intercepted traffic (ties into the packet-capture skill) and note what's readable: cleartext protocols (HTTP, FTP, telnet) hand over credentials; TLS traffic stays encrypted (that's the defence working).
4. **Understand the limits** — properly-validated TLS resists this (the victim gets a certificate warning if the attacker tries to intercept HTTPS); the exposure is mainly unencrypted traffic and users who click through warnings.
5. **Demonstrate impact for the report** (authorised tests) — the value is showing that a flat LAN lets any host intercept any other, to justify the defensive controls below. Don't harvest real user data beyond what proves the point.

### Cheatsheet

```
attack (LAB / AUTHORISED ONLY — affects others' traffic)
  ARP has no auth -> forge replies -> victim & gateway send traffic through you
  bettercap -iface eth0  ->  set arp.spoof.targets ...; arp.spoof on
  enable IP forwarding so the victim stays connected (silent MITM)
  capture with the packet-analysis skill

what's exposed vs protected
  cleartext (HTTP/FTP/telnet)  -> credentials + data readable
  valid TLS (HTTPS)            -> stays encrypted; cert warning on intercept = defence working

DEFENCES (the real point)
  dynamic ARP inspection (DAI) + DHCP snooping on switches
  port security (limit MACs per port)
  802.1X / NAC (only authorised devices on the LAN)
  encrypt everything (TLS everywhere) + HSTS (stops SSL-strip)
  segment the network (limit who shares a broadcast domain)
```

### Reading the situation

- **A flat LAN where any host can ARP-spoof any other** = every device on that segment can intercept the others; a foundational weakness that this attack demonstrates. The fix is switch-level controls plus segmentation.
- **Cleartext protocols in use** = ARP-spoofing hands the attacker credentials and data directly; the presence of HTTP/FTP/telnet turns interception into immediate compromise.
- **Users clicking through TLS certificate warnings** = the one way valid TLS gets defeated; awareness plus HSTS (which prevents the click-through) matters.
- **Switches without DAI/DHCP snooping/port security** = nothing stops the ARP forgery at the network layer; these features are the direct defence.
- **No NAC/802.1X** = any device that plugs in joins the LAN and can attack it; access control keeps unauthorised devices off.

### The defence (the payoff)

- **Enable Dynamic ARP Inspection (DAI) with DHCP snooping** on managed switches — it validates ARP against known MAC/IP bindings and drops the forged replies that make spoofing work. The most direct fix.
- **Port security** — limit MAC addresses per switch port to stop a rogue device impersonating others.
- **802.1X / NAC** — only authenticated, authorised devices get on the network at all (see the NAC skill).
- **Encrypt everything** — TLS everywhere means interception yields ciphertext, and **HSTS** stops the SSL-strip click-through. This blunts the impact even if positioning succeeds.
- **Segment the network** so fewer hosts share a broadcast domain, shrinking who can attack whom (see the segmentation skill).
- **Monitor for ARP anomalies** — sudden MAC/IP binding changes are a spoofing signal.

### Pitfalls

- **Running it outside a lab/authorised scope.** Intercepting other people's traffic is illegal; this attack affects real users. The skill exists to teach defence — keep the offensive part contained.
- **Assuming TLS makes it harmless.** TLS protects content, but cleartext protocols, downgrade attempts, and users clicking through warnings still expose data. Encrypt everything and use HSTS.
- **Defending at the host, not the switch.** The effective controls (DAI, DHCP snooping, port security, NAC) live on the network infrastructure; host-level ARP static entries don't scale.
- **Ignoring segmentation.** Flat networks maximise who can MITM whom; segmentation shrinks the attack surface.

### References

- MITRE ATT&CK — T1557.002 (ARP Cache Poisoning), T1557 (Adversary-in-the-Middle)
- Switch vendor documentation on Dynamic ARP Inspection and DHCP snooping
- The packet-capture, network-segmentation, and network-access-control skills
- CWE-300 (channel accessible by non-endpoint)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.