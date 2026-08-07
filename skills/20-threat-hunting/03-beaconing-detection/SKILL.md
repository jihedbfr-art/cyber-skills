---
name: beaconing-detection
domain: 20-threat-hunting
description: Use when hunting for command-and-control beaconing in network telemetry — the regular-interval callbacks that reveal C2 even when the destination and payload are unknown.
difficulty: intermediate
tags: [threat-hunting, beaconing, c2, network, analysis]
tools: []
---

## Purpose

Compromised hosts phone home, and most C2 does it by *beaconing* — connecting to the controller at regular intervals to check for commands. That regularity is a behavioural signature you can hunt even without knowing the destination or decoding the traffic, which makes beaconing detection one of the highest-value network hunts. This skill covers finding beaconing in network logs, catching C2 that IP/domain blocklists miss.

## When to use it

Hunting network telemetry (proxy, firewall, DNS, netflow) for C2 you don't already have indicators for. It's especially valuable because it targets *behaviour* (durable) rather than infrastructure (which rotates), so it catches C2 using new, unknown domains and IPs.

## Procedure

1. **Get the network connection data.** Beaconing hunts run on connection logs with timestamps and destinations — proxy logs, firewall logs, netflow, or DNS logs. You need enough history to see the pattern (the interval) repeat.
2. **Hunt for regularity, not destination.** The core signal: a host connecting to the same external destination at *consistent intervals* over time. Group connections by source-destination pair and look at the time-between-connections — a tight, repeating interval (every 60s, every hour) is the beacon signature.
3. **Account for jitter.** Modern C2 adds randomisation (jitter) to the interval to evade simple detection, so the interval won't be perfectly constant — hunt for *approximate* regularity (a clustered distribution of intervals) rather than exact. Beaconing analysis tools handle jitter statistically.
4. **Filter out legitimate beaconing — the main challenge.** Lots of benign software beacons: software update checks, telemetry, monitoring agents, synced apps. These produce the same regular-interval pattern. Baseline and allowlist the known-good beacons (a specific update service to a vendor domain) so the malicious ones surface — this filtering is most of the work.
5. **Prioritise the suspicious survivors.** After filtering known-good, focus on beacons to newly-seen/rare destinations, to uncategorised domains, with unusual user-agents, or from hosts that shouldn't be beaconing anywhere. Combine the timing signal with destination reputation and context.
6. **Investigate hits** as a hunt (the hypothesis-driven method) — confirm whether the beaconing host and destination are malicious, pivot to endpoint data, and escalate to IR if confirmed.
7. **Operationalise** — a confirmed beaconing pattern (and the analytic that found it) becomes a detection; the C2 destination becomes an IoC (feeds detection and threat-intel).

## Cheatsheet

```
C2 beacons = connects to controller at REGULAR INTERVALS -> behavioural signature
  hunt the BEHAVIOUR (durable) not the destination (rotates) -> catches unknown C2

data: connection logs with timestamps + destination (proxy/fw/netflow/DNS), enough HISTORY

signal: same src->dst pair at CONSISTENT intervals over time
  group by src-dst -> time-between-connections -> tight repeating interval = beacon

jitter: modern C2 randomises interval -> hunt APPROXIMATE regularity (clustered dist),
        not exact ; beaconing tools handle it statistically

FILTER legit beacons (the main work): updates, telemetry, monitoring, synced apps
  baseline + allowlist known-good -> malicious surface

prioritise survivors: new/rare dest | uncategorised domain | odd user-agent |
  host that shouldn't beacon
investigate -> pivot to endpoint -> escalate ; operationalise (detection + IoC)
```

## Reading the hunt

- **A host beaconing to a rare/new/uncategorised destination at a regular (jittered) interval** = a strong C2 lead; the timing signature plus a suspicious destination is exactly what this hunt targets, and it catches C2 with no known indicators. The highest-value find.
- **Regular beaconing to a known vendor/update service** = benign; these dominate the results and are the noise to filter. Baselining known-good is most of the work — without it, the hunt drowns in legitimate beacons.
- **Beaconing with jitter** (approximately-regular, not exact intervals) = deliberate evasion; don't require perfect regularity or you'll miss modern C2. Hunt the clustered interval distribution.
- **A host beaconing that has no business talking to the internet** (a server that shouldn't, an internal-only system) = suspicious regardless of destination reputation; context sharpens the timing signal.
- **A confirmed beacon** = escalate to IR (active C2), extract the destination as an IoC, and turn the pattern into a detection so you catch it automatically next time.
- **Timing signal + destination reputation + host context combined** = the reliable way to separate malicious beacons from the benign majority.

## Pitfalls

- **Requiring perfect regularity.** Jitter breaks exact-interval detection; modern C2 randomises deliberately. Hunt approximate/clustered regularity, not a constant.
- **Not filtering legitimate beacons.** Benign software beacons constantly (updates, telemetry, monitoring); without baselining known-good, the malicious signal drowns. This filtering is the core effort.
- **Hunting destination instead of behaviour.** Chasing known-bad IPs/domains misses C2 on new infrastructure; the timing behaviour is what's durable and catches unknown C2.
- **Insufficient history.** You need enough time span to see the interval repeat; a short window can't reveal a slow (hourly/daily) beacon.
- **Timing alone.** A regular interval to a benign destination is benign; combine timing with destination reputation and host context to prioritise.

## References

- The malware c2-and-network-analysis, network packet-capture-analysis, and dns-and-proxy-hunting skills
- Beaconing analysis tooling (RITA and similar) and netflow analysis
- MITRE ATT&CK — TA0011 (Command and Control), T1071
- The hypothesis-driven-hunting and operationalising-a-hunt skills
