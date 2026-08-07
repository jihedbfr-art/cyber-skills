---
name: exfiltration-simulation
domain: 16-red-teaming-and-adversary-emulation
description: Use when testing data-loss controls in an authorised engagement — simulating exfiltration with dummy data to see whether the organisation detects and prevents data leaving.
difficulty: advanced
tags: [red-team, exfiltration, dlp, authorized, data-loss]
tools: []
---

## Purpose

The objective of many intrusions is data theft, and the final phase is exfiltration — getting data out of the environment. Simulating exfiltration tests the organisation's data-loss controls and detection: can they see and stop data leaving? This skill covers the exfiltration phase of an authorised engagement, done safely with dummy data (never real sensitive data), and — the defensive core — how exfiltration is detected and prevented. It tests the last line of defence against data breaches.

## When to use it

The objective/final phase of an authorised engagement (RoE) where testing data-loss prevention is in scope. The critical safety rule: simulate with **dummy/canary data**, never actual sensitive data — the point is testing the controls, not exfiltrating real information.

## Procedure (authorised, dummy data)

1. **Use dummy/canary data — the safety rule.** Never exfiltrate real sensitive data (that would be causing the very breach you're testing against). Use marked dummy data or canary tokens that simulate sensitive data and can be tracked. This tests the controls without the risk.
2. **Operate within the RoE**, with the data-handling terms explicit (what data is used, how it's handled and destroyed — the scoping skill covers this).
3. **Understand the exfiltration channels** (conceptual) — how adversaries get data out:
   - **Over C2** — through the existing command-and-control channel.
   - **Over alternative protocols** — DNS tunnelling, HTTPS to cloud storage, email.
   - **To external services** — cloud storage, file-sharing sites, paste sites.
   - **Physical/other** — removable media (where in scope).
4. **Emulate the actor's exfiltration approach** (the emulation-planning skill) and attempt to move the dummy data out via the relevant channels.
5. **Focus on testing detection and prevention — the defensive point.** The questions: does Data Loss Prevention (DLP) detect/block the data leaving? does network monitoring catch the exfiltration channel (DNS tunnelling, large outbound transfers, unusual destinations — the threat-hunting DNS/proxy skill)? does the SOC alert? Data leaving undetected is the finding.
6. **Understand how exfiltration is detected and prevented** (the valuable defensive knowledge):
   - **DLP** — content inspection detecting sensitive data patterns leaving.
   - **Network detection** — DNS tunnelling signatures, large/unusual outbound transfers, connections to file-sharing/cloud-storage destinations (the DNS/proxy hunting and network skills).
   - **Egress controls** — restricting outbound connections (segmentation, egress filtering) so exfiltration channels are limited.
7. **Test multiple channels to gauge coverage depth** — DLP might catch email but miss DNS tunnelling; testing several reveals whether detection is comprehensive or channel-specific.
8. **Report the exfiltration outcome** — "which channels succeeded in moving (dummy) data out undetected, and which were caught" is a critical finding that drives DLP, egress control, and network-detection improvements — the last line of defence against data breaches.

## Cheatsheet

```
many intrusions aim at DATA THEFT ; exfiltration = final phase (data OUT)
  simulate -> test data-loss controls + detection (last line of defence vs breaches)

SAFETY RULE: DUMMY / canary data ONLY, never real sensitive data (that = causing the breach)
  RoE with explicit data-handling + destruction terms

channels (conceptual)
  over C2 | alternative protocols (DNS tunnelling, HTTPS to cloud, email)
  | external services (cloud storage, file-sharing, paste sites) | removable media (if in scope)

emulate actor's approach -> attempt to move dummy data out

DEFENSIVE POINT: test DETECTION + PREVENTION
  DLP (content inspection catches sensitive patterns leaving)
  network detection: DNS tunnelling signatures | large/unusual outbound | file-sharing/cloud destinations
    [dns-and-proxy-hunting, network]
  egress controls (restrict outbound -> limit channels)
TEST MULTIPLE channels (DLP catches email but misses DNS tunnelling?) -> coverage depth
report: which channels succeeded undetected + which caught -> improve DLP/egress/detection
```

## Reading the phase

- **Dummy data exfiltrated undetected** = a data-loss control gap; the last line of defence didn't catch data leaving. A critical finding, since data theft is the objective of many intrusions. Drives DLP, egress, and network-detection improvements.
- **Exfiltration detected/blocked by DLP or network monitoring** = the defensive win; the controls caught data leaving, which is exactly what the phase tests. Note which control caught it and which channel.
- **DLP catching one channel (email) but missing another (DNS tunnelling)** = channel-specific, shallow coverage; testing multiple channels reveals whether detection is comprehensive. A common gap — DNS tunnelling especially evades content-based DLP.
- **Large or unusual outbound transfers not flagged** = a network-detection gap; volume and destination anomalies are catchable signals (the network and hunting skills).
- **Unrestricted egress allowing exfiltration to any destination** = an egress-control gap; restricting outbound connections limits the channels available. Segmentation/egress filtering is the architectural fix.
- **Which channels succeeded undetected and which were caught** = the critical finding; the phase's value is testing and improving the organisation's ability to detect and prevent data loss.

## Pitfalls

- **Using real sensitive data.** The cardinal safety error — exfiltrating real data *is* the breach you're testing against. Use dummy/canary data only, with explicit handling/destruction terms.
- **Testing one channel.** DLP and detection are often channel-specific (catch email, miss DNS tunnelling); test multiple channels to reveal coverage depth. A single-channel test overstates the defence.
- **Focusing on succeeding over the defensive outcome.** The value is which channels are detected/blocked; that's the finding, not whether data got out.
- **Ignoring the egress-control angle.** Detection is one axis; restricting outbound connections (egress filtering, segmentation) limits the channels available and is part of the fix.
- **Providing operational exfiltration tooling.** Conceptual by design; the value is understanding channels and their detection/prevention.

## References

- MITRE ATT&CK — TA0010 (Exfiltration): T1041, T1048 (alternative protocol), T1567 (web service)
- The threat-hunting dns-and-proxy-hunting and network-security domains (detection), and DLP guidance
- Canary tokens (canarytokens.org) for safe exfiltration simulation
- The scoping-and-rules-of-engagement and attack-emulation-planning skills
