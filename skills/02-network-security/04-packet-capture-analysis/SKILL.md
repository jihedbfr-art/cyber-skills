---
name: packet-capture-analysis
domain: 02-network-security
description: Use when you need to read what's actually on the wire — capturing and analysing traffic with tcpdump and Wireshark to investigate an incident, confirm a finding, or debug.
difficulty: intermediate
tags: [network, pcap, tcpdump, wireshark, analysis, forensics]
tools: [tcpdump, wireshark, tshark]
---

## Purpose

When you need ground truth about network activity — is this connection encrypted, what is this host really talking to, is data leaving — you read the packets. Packet capture analysis is the skill of grabbing traffic and making sense of it: confirming a suspected C2 channel, spotting cleartext credentials, or debugging why a connection fails. This skill covers capturing efficiently and analysing without drowning in packets.

## When to use it

Incident investigation (what did this host do on the network), confirming a security finding (is this traffic really unencrypted), threat hunting (beaconing patterns), or debugging. It complements the higher-level logs — packets are the most granular, most truthful source, and the hardest to fake.

## Procedure

1. **Capture with a filter, not everything.** Unfiltered capture on a busy link produces gigabytes fast. Use a capture (BPF) filter to grab only relevant traffic — a host, a port, a protocol:
   ```
   tcpdump -i eth0 host 10.0.0.5 -w capture.pcap          # one host
   tcpdump -i eth0 'port 53 or port 80' -w web-dns.pcap    # specific services
   ```
2. **Capture to a file, analyse offline.** Write to `.pcap` and open in Wireshark for the analysis — live analysis on the wire is error-prone and you can't go back over it.
3. **Get the shape first.** In Wireshark, use Statistics → Conversations and Protocol Hierarchy to see who's talking to whom and what protocols dominate, before diving into individual packets. This orients you fast.
4. **Apply display filters** to narrow to what matters — display filters (Wireshark syntax) are different from capture filters and let you slice the saved capture:
   ```
   ip.addr == 10.0.0.5 && tcp.port == 443
   http.request                 # just HTTP requests
   dns.qry.name contains "suspicious"
   ```
5. **Follow the stream** to reconstruct a conversation — "Follow TCP/HTTP Stream" reassembles a session so you read it as the application saw it (a full HTTP request/response, a cleartext login).
6. **Look for the security-relevant signals** — cleartext credentials, unexpected destinations, beaconing (regular-interval connections to one host), large outbound transfers (exfiltration), and protocol anomalies.
7. **Handle it as evidence** if it's for an incident — capture integrity and chain of custody matter (see the forensics domain).

## Cheatsheet

```bash
# capture (tcpdump) — always filter
tcpdump -i eth0 host 10.0.0.5 -w cap.pcap
tcpdump -i eth0 'tcp port 443' -w tls.pcap
tcpdump -i eth0 -n 'port 53' -w dns.pcap      # -n = no name resolution (faster)

# capture (BPF) filter examples
host 10.0.0.5 | net 10.0.0.0/24 | port 80 | tcp | udp | 'tcp port 443 and host x'

# analyse (Wireshark display filters — different syntax!)
ip.addr == 10.0.0.5
tcp.port == 443
http.request | dns | tls.handshake
tcp.flags.syn == 1 && tcp.flags.ack == 0     # SYNs (scans/connections)
ip.len > 1400                                 # large packets (transfers)

# orient first
Statistics -> Conversations | Protocol Hierarchy
right-click -> Follow -> TCP/HTTP Stream      # reconstruct a session

# command-line analysis
tshark -r cap.pcap -Y "http.request" -T fields -e ip.dst -e http.host
```

## Reading the capture

- **Cleartext credentials** (HTTP Basic auth, FTP, telnet, an unencrypted login in a followed stream) = a real finding — sensitive data on the wire in the clear.
- **Beaconing** (regular-interval connections from a host to one external destination) = a strong C2 signal; the regularity is the tell (ties into the threat-hunting beaconing skill).
- **Unexpected destinations** = a host talking to something it shouldn't — an unknown external IP, an internal system it has no business reaching (lateral movement).
- **Large or sustained outbound transfers** = potential data exfiltration; check the destination and volume.
- **Protocol on a wrong port** (SSH on 443, DNS carrying oddly-large payloads) = tunnelling/evasion; worth investigating.
- **Encrypted traffic you can't read** = expected for TLS — you see metadata (who, when, how much) but not content, which is often enough for the investigation.

## Pitfalls

- **Capturing everything.** Unfiltered capture on a busy link fills the disk and buries the signal. Filter at capture time.
- **Confusing capture and display filters.** BPF (capture) and Wireshark display-filter syntax are different; `port 80` captures, `tcp.port == 80` displays. Using the wrong one in the wrong place returns nothing or everything.
- **Diving into packets before getting the shape.** Start with Conversations/Protocol Hierarchy; individual packets make sense only once you know the overall picture.
- **Expecting to read encrypted content.** TLS traffic shows metadata, not plaintext, without the keys — don't mistake "can't read it" for "nothing there".
- **Ignoring evidence handling.** For incidents, an unmanaged capture may not hold up; preserve integrity (see forensics).

## References

- Wireshark documentation and display-filter reference
- tcpdump / BPF filter documentation
- SANS packet analysis resources
- The threat-hunting (beaconing) and forensics domains
