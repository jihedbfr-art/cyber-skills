---
format: "v2"
name: "certificate-transparency"
title: "Certificate Transparency"
title_fr: "Transparence des certificats"
description: "Use when discovering an organisation's hosts and subdomains through public Certificate Transparency logs — a passive source that reveals names as soon as a certificate is issued."
description_fr: "À utiliser pour découvrir les hôtes et sous-domaines d'une organisation via les journaux publics de Certificate Transparency — une source passive qui révèle les noms dès qu'un certificat est émis."
domain: "01-osint-and-reconnaissance"
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

Every TLS certificate issued by a public CA is recorded in Certificate Transparency (CT) logs — a public, append-only record designed to catch mis-issued certificates. As a side effect, it's a goldmine for recon: the moment an organisation gets a certificate for `staging.example.com`, that name is publicly logged, even if the host is internal-only or never linked anywhere. This skill covers mining CT logs to discover hosts, complementing DNS-based subdomain enumeration.

### When to use it

Early recon alongside subdomain enumeration and DNS recon — CT often reveals names those miss, because a certificate exists before the host is public or DNS is fully configured. Also a strong self-audit: it shows what names your organisation has quietly exposed by issuing certificates.

### Procedure

1. **Query CT logs for the target domain.** `crt.sh` is the simplest interface — it searches the logs for all certificates matching a domain and returns every name they cover:
   ```
   # via web: crt.sh/?q=%25.example.com
   # via API (JSON):
   curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sort -u
   ```
2. **Extract and deduplicate the names** — certificates often cover multiple names (SANs), so one cert can reveal several hosts. Sort unique to get a clean list.
3. **Watch for internal/pre-production names** — CT frequently exposes `dev`, `staging`, `internal`, `vpn`, and one-off hostnames that DNS enumeration and wordlists won't find, because the org got a real cert for them.
4. **Feed the results into resolution and probing** — like subdomain enumeration, resolve the names and check which are live (hand off to the subdomain-enumeration skill's `dnsx`/`httpx` steps). CT gives names; you still confirm what's reachable.
5. **Note wildcard certs** — a `*.example.com` cert doesn't enumerate the specific subdomains but confirms wildcard usage, which affects how you interpret DNS results.
6. **Cross-reference with other sources** — CT complements DNS/passive-DNS enumeration; each finds names the others miss, so merge them.

### Cheatsheet

```bash
curl -s "https://crt.sh/?q=%25.example.com&output=json" \
  | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u


dev/staging/uat/internal/vpn hosts       (cert issued before host is public)
one-off hostnames not in any wordlist
newly-provisioned names (CT logs update as certs are issued)

cat ct-names.txt | dnsx -silent | httpx -silent -title -sc
```

### Reading the output

- **Pre-production / internal hostnames** (`staging`, `dev`, `internal`, `vpn`) = high value; these often have weaker security than production and CT is the source most likely to reveal them. A staging box with a real cert is a classic soft target.
- **Names not found by DNS brute-forcing** = CT's edge — it reveals names that were never in a wordlist because the org simply issued a certificate for them.
- **A newly-appearing name** = CT logs update as certs are issued, so monitoring them surfaces new infrastructure as it comes online (useful for continuous recon).
- **Wildcard certificates (`*.example.com`)** = don't enumerate specific hosts but confirm wildcard use, which explains why DNS resolves arbitrary names.
- **A dead name from an old cert** = CT is historical; a logged name may no longer resolve. Resolve before acting.

### The fix / defensive use

CT is public and can't be opted out of (that's the point — it's a security mechanism against mis-issuance), so the defensive angle is awareness and hygiene:

- **Monitor CT for your own domains** — services alert you when a certificate is issued for your domain, which catches both your own forgotten hosts and *mis-issued or rogue certificates* an attacker obtained for your name (the original purpose of CT).
- **Don't rely on obscurity** — if you issue a public cert for an internal host, its name is now public. Secure those hosts as if their names are known, because they are.
- **Consider wildcard certs** for genuinely internal name spaces so individual hostnames aren't logged (a trade-off — wildcards have their own key-management downsides).
- **Inventory from CT** — use it to find hosts you forgot you exposed, and decommission or secure them.

### Pitfalls

- **Skipping CT because you ran DNS enumeration.** They find different things; CT reveals names no wordlist contains. Use both and merge.
- **Trusting names as live.** CT is a historical log; a name may be from an expired cert on a decommissioned host. Resolve and probe before acting.
- **Forgetting the defensive value.** CT monitoring catches rogue certificates issued for your domain — a real attack signal, not just a recon source.
- **Assuming an internal name is safe because it's "not published".** A public certificate publishes it. Secure the host accordingly.

### References

- crt.sh, Certspotter, and Censys certificate search
- Certificate Transparency project (certificate-transparency.org) — RFC 6962
- OWASP WSTG-INFO — enumerate infrastructure
- The subdomain-enumeration skill (CT is one of its passive sources)

## Inputs
- Relevant source code, logs, network traces, or system specifications.

## Outputs
- Analysis findings, security audit report, or generated code artifacts.