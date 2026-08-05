---
name: tls-configuration
domain: 14-cryptography-and-pki
description: Use when configuring TLS on a server — protocol versions, cipher suites, and settings that score clean and stay usable — the config side of the network TLS-inspection check.
difficulty: intermediate
tags: [crypto, tls, ssl, ciphers, server-config]
tools: []
---

## Purpose

TLS-inspection (network domain) tells you what a service's TLS *looks like* from the outside; this skill is the other half — how to *configure* a server so it comes out clean and stays compatible with real clients. Getting TLS config right is mostly about disabling the old and dangerous while keeping what legitimate clients need, and not hand-picking cipher lists you don't fully understand. This skill covers the server-side settings and the sane way to derive them.

## When to use it

Configuring TLS on any server (web, API, mail, load balancer, reverse proxy), or hardening an existing config flagged by a scan. It pairs directly with the network `tls-inspection` skill (which verifies the result from the wire).

## Procedure

1. **Don't hand-write cipher strings — generate them.** The reliable way to get a correct, current config is a trusted generator (Mozilla SSL Configuration Generator) for your server software and a chosen compatibility profile. Hand-picked cipher lists are where subtle mistakes and outdated suites creep in.
2. **Pick a compatibility profile deliberately:**
   - **Modern** — TLS 1.3 (and 1.2) only, strongest ciphers; use when you don't need to support old clients.
   - **Intermediate** — TLS 1.2 + 1.3 with a broad-but-safe cipher set; the sensible default for general-purpose services.
   - **Old** — only if you genuinely must support legacy clients, and understand the weakened security.
3. **Enable only TLS 1.2 and 1.3.** Disable SSLv2/3 and TLS 1.0/1.1 — deprecated and attackable (POODLE, BEAST). TLS 1.3 is preferred (faster, forward-secret by design, fewer footguns).
4. **Require forward secrecy** — ECDHE key exchange, so past traffic can't be decrypted if the private key later leaks. AEAD cipher suites (AES-GCM, ChaCha20-Poly1305) only; no RC4, 3DES, or non-FS suites.
5. **Configure the surrounding settings** — a valid certificate with a complete chain (see certificate-management), OCSP stapling, and HSTS for web services (the security-headers skill). These complete the picture beyond the cipher list.
6. **Verify from the outside** with the tls-inspection tools (`testssl.sh`/SSLyze) after configuring — the config is a claim; the wire is the truth. Confirm the score and that no weak protocols/ciphers slipped through.
7. **Balance security against your clients.** The Modern profile is strongest but breaks old clients; choose the profile that fits your actual client base rather than blindly maximising and locking legitimate users out (while still dropping genuinely obsolete protocols).

## Cheatsheet

```
generate, don't hand-write
  Mozilla SSL Configuration Generator -> your server + a profile
  profiles: Modern (TLS1.3/1.2, strongest) | Intermediate (default) | Old (legacy only)

core config
  protocols: TLS 1.2 + 1.3 ONLY  (no SSLv2/3, no TLS 1.0/1.1)
  ciphers:   AEAD + forward secrecy (ECDHE) ; no RC4/3DES/non-FS
  prefer TLS 1.3 (FS by design, fewer footguns)

surrounding
  valid cert + complete chain (certificate-management)
  OCSP stapling ; HSTS for web (security-headers)

verify from the wire: testssl.sh / sslyze  (config = claim, wire = truth)
balance: pick the profile that fits your real clients (don't lock them out)
```

## Reading the config/result

- **SSLv3 / TLS 1.0 / 1.1 enabled** = deprecated protocols enabling downgrade attacks; disable them. A clear finding regardless of client concerns.
- **Non-forward-secret or weak ciphers** (RC4, 3DES, static RSA key exchange) = decryptable/interceptable; restrict to AEAD + ECDHE.
- **A hand-written cipher list with an outdated or mis-ordered suite** = the subtle-mistake risk; regenerate from a trusted source instead.
- **Modern profile breaking legitimate old clients** = over-hardening for your client base; step to Intermediate (still dropping obsolete protocols) rather than losing users.
- **Config claims clean but the wire shows a weak suite** = something didn't apply (a default, a fronting proxy); the external scan is what confirms reality.
- **Intermediate/Modern profile, TLS 1.2+1.3, AEAD+FS, valid cert, HSTS, verified externally** = the clean, usable state.

## The fix / best practice

- **Generate the config from a trusted source** (Mozilla generator) for your server and a chosen profile — don't hand-assemble cipher strings.
- **Default to the Intermediate profile**; use Modern where you can drop legacy clients, Old only when truly forced.
- **TLS 1.2 + 1.3 only, AEAD ciphers, forward secrecy required** — disable everything older/weaker.
- **Complete the setup** with a valid chained certificate, OCSP stapling, and HSTS for web.
- **Verify from the wire** with tls-inspection tools after every change, and re-check periodically (recommendations and your config both drift).
- **Match the profile to your real clients** so hardening doesn't lock out legitimate traffic.

## Pitfalls

- **Hand-writing cipher lists.** Easy to include an outdated suite or misorder preferences; generate from a trusted source instead.
- **Leaving old protocols on.** TLS 1.0/1.1 and SSLv3 enable downgrade attacks; disable them even when keeping some client compatibility.
- **Over-hardening past your clients.** The Modern profile breaks legacy clients; if that's a problem, use Intermediate rather than losing users — but still drop obsolete protocols.
- **Trusting the config without verifying.** A fronting proxy, a default, or a load balancer can serve weaker TLS than your config says; verify from the wire.
- **Config without the surrounding pieces.** A perfect cipher list with an expired cert or no HSTS is still a finding; certificate and header hygiene matter too.

## References

- Mozilla SSL Configuration Generator and Server Side TLS guidelines
- OWASP Transport Layer Protection Cheat Sheet
- The network tls-inspection skill (external verification) and certificate-management skill
- NIST SP 800-52 (TLS configuration guidance)
