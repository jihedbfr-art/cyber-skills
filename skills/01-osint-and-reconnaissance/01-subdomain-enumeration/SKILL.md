---
name: subdomain-enumeration
domain: 01-osint-and-reconnaissance
description: Use when you need to map an organisation's subdomains to find hosts and services outside the obvious www — the first recon step that feeds web, API, and cloud testing.
difficulty: beginner
tags: [osint, recon, dns, attack-surface, passive]
tools: [subfinder, amass, dnsx, httpx]
---

## Purpose

Organisations expose far more than their main site. Staging boxes, old admin panels, forgotten marketing microsites, dev APIs — they all live on subdomains, and the neglected ones are usually the soft spot. This skill gets you a deduplicated, live list of an org's subdomains so the rest of your recon has somewhere to point.

It stops at "here are the hosts that answer." Fingerprinting and vulnerability testing are separate skills.

## When to use it

At the start of almost any external assessment, right after you've confirmed the root domains are in scope. Also useful for continuous monitoring — running it weekly surfaces new hosts as an org ships them.

Skip it if you were handed an explicit host list and told not to expand beyond it. Enumeration widens scope, and scope is contractual.

## Procedure

1. Confirm the root domain(s) are inside your authorised scope. Wildcards like `*.example.com` usually cover subdomains; if you're unsure, ask before you enumerate.
2. Passive first. Pull names from public sources (CT logs, passive DNS, search engines) without sending a packet to the target:
   ```
   subfinder -d example.com -all -silent -o passive.txt
   ```
3. Add a second source and merge — no single tool has everything:
   ```
   amass enum -passive -d example.com -o amass.txt
   sort -u passive.txt amass.txt > all-names.txt
   ```
4. Resolve the list to drop dead names and keep what actually has DNS records:
   ```
   dnsx -l all-names.txt -silent -a -resp -o resolved.txt
   ```
5. Check which resolved hosts serve HTTP/HTTPS, and grab their titles and status codes:
   ```
   httpx -l resolved.txt -title -status-code -tech-detect -o live.txt
   ```
6. Only if active enumeration is in scope, brute-force with a wordlist to catch names not in any public source. This sends traffic to the target's DNS — treat it as active recon:
   ```
   dnsx -d example.com -w subdomains.txt -silent -o bruteforced.txt
   ```

## Cheatsheet

```bash
# fast passive sweep
subfinder -d example.com -all -silent | dnsx -silent -a | httpx -silent -title -sc

# multiple roots from a file
subfinder -dL roots.txt -all -silent -o subs.txt

# resolve + probe in one pass
cat subs.txt | dnsx -silent | httpx -silent -status-code -title -tech-detect

# find only the interesting status codes
httpx -l resolved.txt -mc 200,401,403 -title
```

## Reading the output

A live host with a `200` and a login title is a lead. A `401`/`403` is often more interesting — something's there and it's trying to hide. Look for:

- hostnames with `dev`, `staging`, `test`, `uat`, `old`, `internal` — lower-effort security, same data
- unexpected technologies in the `tech-detect` column (a lone WordPress box in an otherwise custom stack)
- hosts resolving to cloud IPs or, better, to a CNAME pointing at a service that no longer exists — that's a subdomain takeover lead

## Pitfalls

- **Scope creep.** CT logs return names for domains you may not be allowed to touch (shared infra, acquired companies). Filter to your authorised roots.
- **Stale results.** A name in a CT log from 2019 may be long dead. Always resolve before you act on a list.
- **Rate limits and API keys.** Passive sources throttle hard without keys. Configure them or you'll get a fraction of the real picture and think the surface is small.
- **Wildcard DNS.** Some domains resolve *everything* to one IP. `dnsx` can filter wildcards; without that you'll chase hundreds of phantom hosts.

## References

- OWASP WSTG — Information Gathering (WSTG-INFO-04, enumerate applications on the web server)
- ProjectDiscovery docs for subfinder, dnsx, httpx
- OWASP Amass user guide
