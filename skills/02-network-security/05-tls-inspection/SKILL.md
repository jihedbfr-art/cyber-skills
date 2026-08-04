---
name: tls-inspection
domain: 02-network-security
description: Use when checking a service's TLS configuration on the wire — protocol versions, cipher suites, and certificate validity — to find weak crypto and misconfiguration.
difficulty: beginner
tags: [network, tls, ssl, ciphers, certificates]
tools: [testssl, sslyze, nmap]
---

## Purpose

TLS protects data in transit — but only if it's configured well. Old protocol versions, weak cipher suites, and expired or misissued certificates leave connections vulnerable to downgrade, interception, and decryption. This skill covers inspecting a service's TLS from the outside to find those weaknesses, and the configuration that scores clean while staying usable. It's the wire-level check that complements the crypto domain's TLS-configuration guidance.

## When to use it

Assessing any TLS-enabled service (HTTPS, mail, VPN, database over TLS, APIs), or verifying your own endpoints after a config change. Fast, non-intrusive, and high-signal — TLS misconfiguration is common and directly relevant to data-in-transit security.

## Procedure

1. **Scan the endpoint's full TLS configuration.** A dedicated tool enumerates supported protocols, cipher suites, certificate details, and known vulnerabilities in one pass:
   ```
   testssl.sh https://example.com:443
   # or
   sslyze example.com:443
   ```
2. **Check protocol versions.** Only TLS 1.2 and 1.3 should be enabled. SSLv2/SSLv3 and TLS 1.0/1.1 are deprecated and weak — their presence is a finding. TLS 1.3 is preferred where supported.
3. **Check cipher suites.** Weak or obsolete ciphers (RC4, DES/3DES, export-grade, NULL, anything without forward secrecy) should be disabled. Prefer AEAD suites (AES-GCM, ChaCha20-Poly1305) with forward secrecy (ECDHE).
4. **Check the certificate.** Valid (not expired, not self-signed for public services), correct hostname (matches the domain and SANs), trusted chain (complete, no missing intermediates), and a strong signature (SHA-256+, RSA-2048+/ECDSA). An expired or mismatched cert is a common, user-facing finding.
5. **Check for known TLS vulnerabilities** the scanner flags — Heartbleed, ROBOT, weak DH parameters, insecure renegotiation, BEAST/POODLE (from old protocols). These map to specific misconfigurations to fix.
6. **Verify hardening extras** — HSTS (from the security-headers skill) for web, OCSP stapling, and secure renegotiation.

## Cheatsheet

```bash
# full TLS assessment
testssl.sh https://example.com          # thorough, human-readable, flags vulns
sslyze example.com:443                   # fast, scriptable
nmap --script ssl-enum-ciphers -p 443 example.com   # cipher/protocol enum

# what "good" looks like
protocols   TLS 1.2 + TLS 1.3 only  (no SSLv2/3, no TLS 1.0/1.1)
ciphers     AEAD (AES-GCM, ChaCha20-Poly1305) + forward secrecy (ECDHE)
            NO RC4 / 3DES / DES / export / NULL / non-FS suites
cert        valid dates, hostname matches SAN, trusted chain, SHA-256+, RSA-2048+
extras      HSTS, OCSP stapling, secure renegotiation

known-vuln flags to act on
  Heartbleed | ROBOT | weak DH | insecure renegotiation | BEAST/POODLE (old TLS)
```

## Reading the output

- **SSLv3 / TLS 1.0 / 1.1 enabled** = deprecated protocols supporting downgrade and known attacks (POODLE, BEAST). Disable them — a clear finding.
- **Weak ciphers offered** (RC4, 3DES, export, NULL, non-forward-secret) = decryptable or interceptable connections; disable and keep only strong AEAD+FS suites.
- **An expired, self-signed (for public), or hostname-mismatched certificate** = a user-facing and trust-breaking finding; often also trains users to click through warnings.
- **A flagged TLS vulnerability** (Heartbleed, ROBOT, weak DH) = a specific, high-severity issue mapping to a concrete fix; prioritise these.
- **No forward secrecy** = past traffic is decryptable if the private key is ever compromised. Enable ECDHE suites.
- **TLS 1.2+1.3 only, strong AEAD+FS ciphers, valid cert, no flagged vulns** = the clean state.

## The fix

- **Disable old protocols** — support only TLS 1.2 and 1.3; remove SSLv2/3 and TLS 1.0/1.1.
- **Restrict to strong cipher suites** with forward secrecy and AEAD (AES-GCM, ChaCha20-Poly1305); disable RC4, 3DES, export, and NULL. Use a current recommended cipher list rather than hand-picking.
- **Fix the certificate** — renew before expiry (automate it), ensure the chain is complete, the hostname/SANs match, and the signature is strong. Automate renewal to prevent the recurring "cert expired" outage.
- **Address flagged vulnerabilities** at their root (patch for Heartbleed, disable RSA key exchange for ROBOT, strong DH params).
- **Add HSTS** for web services and enable OCSP stapling and secure renegotiation.
- **Re-scan after changes** and monitor expiry continuously — TLS config drifts and certs expire on a clock.

## Pitfalls

- **Chasing a perfect score at the cost of compatibility.** Disabling TLS 1.2 or all but the newest ciphers can break legitimate older clients; balance hardening against your actual client base (but old protocols like TLS 1.0/1.1 should still go).
- **Letting certs expire.** The most common, most embarrassing TLS incident — automate renewal and monitor expiry.
- **Missing forward secrecy.** A valid cert and modern protocol still leave past traffic decryptable without FS; require ECDHE.
- **Checking only HTTPS.** TLS protects mail, VPNs, databases, and APIs too — inspect all TLS services, not just the web server.
- **One-time check.** Config drifts and new vulnerabilities emerge; re-scan periodically.

## References

- testssl.sh and SSLyze documentation
- Mozilla SSL Configuration Generator (recommended cipher lists)
- OWASP Transport Layer Protection Cheat Sheet
- The crypto-and-pki TLS-configuration skill and the security-headers (HSTS) skill
