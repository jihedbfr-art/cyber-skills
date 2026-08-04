---
name: vpn-security
domain: 02-network-security
description: Use when assessing or hardening a VPN — IPsec or WireGuard configuration, authentication, and the exposure a remote-access gateway creates — so the tunnel doesn't become the way in.
difficulty: intermediate
tags: [network, vpn, ipsec, wireguard, remote-access]
tools: [ike-scan, nmap]
---

## Purpose

A VPN extends your trusted network to remote users — which makes the VPN gateway a high-value target and a common breach entry point. Weak authentication, outdated VPN software with known CVEs, or a tunnel that drops users straight into a flat internal network turns remote access into attacker access. This skill covers assessing a VPN's configuration and exposure, and hardening it so the tunnel protects rather than exposes.

## When to use it

Assessing a remote-access VPN (the gateway is internet-facing, so it's in scope for external recon and testing), or hardening your own. VPN gateways have been the initial access vector in many major breaches, which makes this high-priority.

## Procedure

1. **Identify the VPN and its version — the first, highest-value check.** VPN appliances (Fortinet, Pulse/Ivanti, Citrix, Palo Alto, Cisco) have had a steady stream of critical, actively-exploited CVEs. An outdated gateway exposed to the internet is frequently the breach. Fingerprint it and check the version against known vulnerabilities:
   ```
   # gateway is internet-facing — Shodan/recon often reveals product + version
   nmap -sV -p 443,500,4500 <vpn-gateway>
   ike-scan <vpn-gateway>            # IKE/IPsec fingerprinting
   ```
2. **Assess authentication.** Is it single-factor (password only) or does it require MFA? A VPN with password-only auth plus leaked credentials (see the OSINT credential-leaks skill) is a straight path in. MFA on the VPN is essential.
3. **Check the protocol and crypto.** For IPsec, weak IKE settings (aggressive mode with PSK, weak DH groups, weak ciphers) are findings. Prefer modern configurations; WireGuard is a strong, simpler modern option. Legacy protocols (PPTP) are broken — flag them.
4. **Check what the tunnel grants access to.** This is the segmentation angle: does a connected VPN user land in a flat internal network (reach everything) or a restricted segment with least-privilege access to only what they need? A VPN into a flat network means one compromised remote credential reaches the crown jewels.
5. **Check split-tunnel vs full-tunnel** and its implications, and confirm the gateway logs connections and authentication attempts for detection.
6. **Test for the known exposure classes** the specific appliance is prone to (within scope) — auth bypass, path traversal, and the current CVEs for that product.

## Cheatsheet

```bash
# 1. fingerprint + version (the #1 risk: outdated exploited appliance)
nmap -sV -p 443,500,4500 gateway
ike-scan -M gateway                  # IKE mode, ciphers, DH groups
# check the product/version against current CVEs (KEV catalog)

# assess
auth        MFA required? or password-only? (password-only + leaked creds = in)
protocol    IPsec (avoid aggressive-mode PSK, weak DH) | WireGuard (modern)
            PPTP = broken, flag it
crypto      strong ciphers + DH groups, no legacy
access      does the tunnel land in a FLAT net or a restricted segment?
logging     connections + auth attempts logged for detection

VPN gateways = frequent breach entry point -> patch fast, MFA, segment
```

## Reading the assessment

- **An outdated VPN appliance with a known/exploited CVE** = the highest-priority finding by far; these are among the most common breach entry points and are actively scanned for. Patch immediately or take it offline.
- **Password-only authentication** = one leaked or phished credential grants network access. MFA is the difference between a stolen password and a breach here.
- **Weak IPsec/IKE settings** (aggressive mode PSK, weak DH, legacy ciphers) or **PPTP** = interceptable or crackable tunnels. Modernise the config.
- **A tunnel into a flat internal network** = a compromised remote credential reaches everything; the VPN undoes your internal segmentation. Restrict what VPN users can reach.
- **No logging on the gateway** = you can't detect a compromised VPN account or an exploitation attempt — a blind spot on a prime target.
- **Patched appliance + MFA + strong crypto + segmented access + logging** = the hardened state.

## The fix

- **Keep the VPN gateway patched — urgently.** Given how heavily these are targeted, treat VPN appliance patches as emergency priority, and watch the KEV catalog for the product. This single practice prevents most VPN breaches.
- **Require MFA** on VPN authentication, ideally phishing-resistant (ties into the MFA skill) — it defeats credential-based access even when passwords leak.
- **Use strong, modern crypto** — a well-configured IPsec (main mode, strong DH, AEAD) or WireGuard; retire PPTP and weak IKE settings.
- **Segment what the tunnel reaches** — VPN users land in a restricted zone with least-privilege access, not the full internal network (per the segmentation skill).
- **Log and monitor** VPN connections and auth attempts; alert on anomalies (impossible travel, connections from a compromised account).
- **Minimise exposure** — restrict the management interface, and disable unused VPN features/protocols.

## Pitfalls

- **Neglecting appliance patching.** VPN gateways are among the most exploited internet-facing systems; a delayed patch is how many breaches start. Patch fast.
- **Password-only VPN auth.** The single most impactful weakness after patching — add MFA.
- **VPN into a flat network.** The tunnel bypasses your internal segmentation, so a remote compromise reaches everything. Restrict and segment VPN access.
- **Legacy protocols/config.** PPTP and weak IKE are broken or crackable; modernise.
- **No monitoring on a prime target.** A compromised VPN account is invisible without logging.

## References

- CISA Known Exploited Vulnerabilities catalog (VPN appliance CVEs feature heavily)
- NIST SP 800-77 (Guide to IPsec VPNs), WireGuard documentation
- The MFA, network-segmentation, and OSINT credential-leaks skills
- CWE-287 (improper authentication), CWE-1188 (insecure default)
