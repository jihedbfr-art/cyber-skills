---
name: certificate-management
domain: 14-cryptography-and-pki
description: Use when managing TLS/PKI certificates across their lifecycle — issuance, renewal, revocation, and inventory — so certificates don't expire in production or become an unmanaged sprawl.
difficulty: intermediate
tags: [crypto, pki, certificates, renewal, lifecycle]
tools: [certbot, openssl]
---

## Purpose

Certificates are the identity layer of TLS, and the most common certificate incident is embarrassingly simple: one expired in production and took a service down. Beyond expiry, unmanaged certificate sprawl means you don't know what you have, what's expiring, or what's misissued. This skill covers managing certificates across their lifecycle — issuance, renewal, revocation, and inventory — so they're an automated non-event instead of a recurring outage.

## When to use it

Running any service that uses certificates (nearly all of them), or fixing a "the cert expired again" pattern. It's operational rather than deeply cryptographic, and that's the point — most certificate problems are lifecycle failures, not crypto failures.

## Procedure

1. **Automate renewal — the single most important practice.** The overwhelming majority of certificate incidents are expiries that automation would have prevented. Use ACME (Let's Encrypt/certbot) or your CA's automated issuance so certificates renew and deploy without a human remembering:
   ```
   certbot renew        # automated ACME renewal (run on a timer)
   ```
2. **Keep an inventory of every certificate** — where it's deployed, its CA, expiry date, and owner. You can't manage what you can't see; certificate sprawl (certs on load balancers, services, internal systems nobody tracks) is where surprise expiries hide. Certificate Transparency logs (OSINT ct skill) can help discover your own public certs.
3. **Monitor expiry and alert well in advance** — even with automation, monitor expiry dates and alert with enough lead time to fix a failed renewal before it becomes an outage. Automation plus monitoring is belt-and-suspenders for a high-impact failure.
4. **Manage issuance properly** — use the right validation level, correct hostnames/SANs, strong keys (RSA-2048+/ECDSA), and a complete chain (missing intermediates is a common "works in my browser, breaks elsewhere" issue). Protect the private keys (key-management skill).
5. **Know your revocation path** — if a private key is compromised or a cert is misissued, you need to revoke and reissue. Understand OCSP/CRL, and that revocation checking is imperfect, so short-lived certificates (which automation enables) reduce the window a compromised cert stays valid.
6. **For internal PKI**, manage the CA carefully — the internal CA's key is a high-value target (it can issue trusted certs for anything), and internal certs need the same lifecycle discipline as public ones.
7. **Prefer short-lived, automated certificates** — they limit exposure from key compromise and force the automation that prevents expiry incidents.

## Cheatsheet

```
the #1 fix: AUTOMATE RENEWAL (most cert incidents = preventable expiry)
  ACME / certbot / CA auto-issuance -> renew + deploy without a human

inventory (can't manage what you can't see)
  every cert: location, CA, expiry, owner ; find sprawl (LBs, services, internal)
  discover public certs via Certificate Transparency (OSINT ct skill)

monitor + alert on expiry with LEAD TIME (backstop for failed auto-renewal)

issuance done right
  correct hostnames/SANs, strong key (RSA-2048+/ECDSA), COMPLETE chain
  (missing intermediates = works here, breaks there), protect private keys

revocation: know OCSP/CRL path for compromise/misissuance
  revocation checking is imperfect -> SHORT-LIVED certs limit the window

internal PKI: the CA key is high-value (can issue anything) — guard it
```

## Reading the situation

- **Manual renewal** = the leading cause of certificate outages; a human will eventually forget. Automate it — this alone prevents most incidents.
- **No certificate inventory** = surprise expiries and misissued certs go unnoticed; you can't monitor what you don't know exists. Sprawl on load balancers and internal systems is the usual blind spot.
- **No expiry monitoring** = even automated renewals can fail silently, and without alerting the first sign is an outage. Monitor with lead time.
- **An incomplete chain (missing intermediates)** = intermittent failures ("works in Chrome, fails for this client"); a common, confusing issuance mistake.
- **Long-lived certs with no revocation plan** = a compromised key stays trusted for a year; short-lived automated certs shrink that window.
- **Automated renewal + full inventory + expiry monitoring + short-lived certs** = certificates become a non-event.

## The fix / best practice

- **Automate issuance and renewal** (ACME/certbot/CA automation) — the highest-impact practice, preventing the expiry incidents that dominate certificate problems.
- **Maintain a complete inventory** with location, expiry, and owner; discover sprawl and public certs (CT logs).
- **Monitor and alert on expiry with lead time** as a backstop even when automated.
- **Issue correctly** — right SANs, strong keys, complete chain — and protect private keys (key-management).
- **Have a revocation/reissue path** and prefer short-lived certificates to limit compromise windows.
- **Guard internal CA keys** and apply the same lifecycle discipline to internal certs.

## Pitfalls

- **Manual renewal.** The classic outage cause — someone forgets, the cert expires, the service goes down. Automate.
- **No inventory.** Untracked certs (load balancers, internal services) expire or get misissued unnoticed. Discover and track them all.
- **No expiry monitoring.** Automation can fail; without alerting you find out via outage. Monitor with lead time regardless.
- **Incomplete certificate chain.** Missing intermediates cause intermittent, hard-to-diagnose failures. Serve the full chain.
- **Long-lived certs, no revocation plan.** A compromised key stays trusted; short-lived automated certs and a known revocation path limit the damage.

## References

- Let's Encrypt / ACME and certbot documentation
- CA/Browser Forum baseline requirements; NIST SP 800-57 (key/cert management)
- The tls-configuration, key-management, and OSINT certificate-transparency skills
- CWE-298 (improper validation of certificate expiration), CWE-295 (improper cert validation)
