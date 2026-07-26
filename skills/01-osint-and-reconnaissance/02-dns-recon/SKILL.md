---
name: dns-recon
domain: 01-osint-and-reconnaissance
description: Use when you want to pull records, zone data, and infrastructure hints out of a target's DNS alone — the quiet recon step before you touch a single host.
difficulty: beginner
tags: [osint, dns, recon, passive]
tools: [dig, dnsx, dnsrecon]
---

## Purpose

DNS tells you where things live before you ever connect. Mail servers, cloud providers, old hosts nobody cleaned up, sometimes a whole zone if the server is misconfigured. This skill walks the record types worth pulling and what each one gives away.

## When to use it

Early recon, right alongside subdomain enumeration. It's passive against public resolvers, so it's safe to run before active scanning is authorised — you're asking the internet's phone book, not knocking on doors.

## Procedure

1. Pull the core records for the domain. Each answers a different question — who hosts mail, who's authoritative, what the apex points at:
   ```
   dig example.com ANY +noall +answer
   dig example.com MX +short
   dig example.com NS +short
   dig example.com TXT +short
   ```
2. Read the TXT records carefully. SPF, DKIM, DMARC, and verification tokens for third-party services (`google-site-verification`, `atlassian-`, etc.) quietly reveal which SaaS the org uses.
3. Try a zone transfer against each authoritative nameserver. It should fail — when it doesn't, you get the entire zone in one request:
   ```
   dig AXFR example.com @ns1.example.com
   ```
4. Check for wildcard DNS before you trust any brute-force list, or you'll chase ghosts:
   ```
   dig random-does-not-exist-12345.example.com +short
   ```
5. Reverse-map the IP ranges you've found to tie hosts back to the org and spot shared infrastructure:
   ```
   dig -x 203.0.113.10 +short
   ```

## Cheatsheet

```bash
dig example.com MX +short           # mail infra
dig example.com NS +short           # authoritative servers
dig example.com TXT +short          # spf/dkim/dmarc, saas tokens
dig AXFR example.com @ns1...         # zone transfer (should fail)
dig -x 203.0.113.10 +short           # reverse lookup
dnsrecon -d example.com -a           # automated sweep incl. AXFR attempt
dnsx -l names.txt -cname -resp        # resolve + show CNAME targets
```

## Reading the output

- A **successful AXFR** is a finding on its own — the nameserver hands you every record, a full map of the internal naming.
- **CNAMEs pointing at a dead SaaS target** (a storage bucket or app that no longer exists) is a subdomain-takeover lead.
- **SPF/DMARC missing or `~all`/`?all`** means the domain is spoofable — flag it for the phishing-defence work.
- **TXT verification tokens** enumerate the org's third-party stack without touching any of it.

## Pitfalls

- **Wildcard DNS** makes every name resolve. Confirm it exists before brute-forcing, or filter wildcards in your resolver.
- **Cached vs live.** Public resolvers cache; query the authoritative server directly (`@ns1...`) when you need current truth.
- **Assuming AXFR is always closed.** It usually is — but it costs one query to check, and the payoff when it's open is the whole zone.

## References

- OWASP WSTG-INFO-02 (Fingerprint, DNS)
- RFC 1035, RFC 7208 (SPF)
- dnsrecon and dnsx documentation
