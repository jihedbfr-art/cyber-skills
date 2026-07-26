---
name: security-headers
domain: 03-web-application-security
description: Use when reviewing or hardening a web app's HTTP response headers — CSP, HSTS, and the rest — knowing which actually reduce risk and which are theatre.
difficulty: beginner
tags: [owasp, headers, csp, hsts, hardening, web]
tools: [curl, burp]
---

## Purpose

Security headers are defence-in-depth: they don't fix vulnerabilities, they shrink the blast radius when one exists. A good CSP turns many XSS bugs into non-events; HSTS closes the downgrade window. This skill covers which headers earn their place, how to check them, and how to set them without breaking the app.

## When to use it

During a web assessment (quick win to check) and during hardening. Fair warning: header scanners hand out low-severity findings generously — this skill is about knowing which matter so you don't drown a report in noise.

## Procedure

1. Pull the response headers and read what's there — and what's missing:
   ```
   curl -sI https://app.tld | grep -iE 'content-security|strict-transport|x-frame|x-content|referrer|permissions'
   ```
2. Assess the four that carry real weight first:
   - **Content-Security-Policy** — the big one. Restricts where scripts/resources load from; a strict policy blunts XSS.
   - **Strict-Transport-Security** — forces HTTPS, closing SSL-strip. Needs a long `max-age` and, ideally, `includeSubDomains; preload`.
   - **X-Frame-Options / CSP frame-ancestors** — stops clickjacking by controlling who can frame the page.
   - **X-Content-Type-Options: nosniff** — stops MIME sniffing that turns an upload into script.
3. For CSP specifically, judge the *content*, not just its presence. A policy with `unsafe-inline` and `unsafe-eval` on `script-src`, or a `default-src *`, is close to no policy at all.
4. Note the ones that are now largely obsolete or misunderstood so you don't over-report them: `X-XSS-Protection` is deprecated (turn it off, `0`), and `Server`/`X-Powered-By` version banners are info leaks worth removing but not vulnerabilities.
5. Verify HSTS won't lock users out before recommending `preload` — once preloaded, HTTPS is mandatory for the whole domain and reversal is slow.

## Cheatsheet

```bash
# see all response headers
curl -sI https://app.tld

# strong baseline to aim for
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'; frame-ancestors 'none'; base-uri 'self'
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), camera=(), microphone=()
X-Frame-Options: DENY        # legacy; frame-ancestors in CSP supersedes it
```

## Reading the output

- **No CSP, or a CSP with `unsafe-inline`/`unsafe-eval`/wildcards** — the meaningful finding. It means an XSS bug gets full impact.
- **Missing or short HSTS** — users are exposed to downgrade on first/again visits. `max-age` under a few months is weak.
- **Framing allowed** (no `X-Frame-Options` and no `frame-ancestors`) — clickjacking is on the table for sensitive actions.
- **Present-but-weak beats absent-and-scary in reports:** rank by real risk. A missing `Permissions-Policy` is minor; a wildcard CSP is not.

## The fix

Set the four load-bearing headers at the edge (reverse proxy or framework middleware) so every response carries them:

- Build the **CSP** iteratively — start in `Content-Security-Policy-Report-Only` mode, collect violation reports, tighten until you can drop `unsafe-inline` (move inline scripts to files or use nonces/hashes), then enforce.
- **HSTS** with a two-year `max-age` and `includeSubDomains`; add `preload` only once you're certain every subdomain does HTTPS.
- `nosniff` and `frame-ancestors 'none'` (or a tight allowlist) — cheap, no downside for most apps.
- Strip `Server`/`X-Powered-By` version details.

## Pitfalls

- **CSP set-and-forget.** A copied policy with `unsafe-inline` gives false comfort. If it doesn't remove inline script, it doesn't stop much XSS.
- **HSTS preload regret.** Preloading before subdomains support HTTPS locks people out. Confirm coverage first.
- **Over-reporting.** A wall of "missing header" lows buries the one that matters. Lead with CSP quality and HSTS.
- **Headers as a substitute for fixes.** They reduce impact; they don't remove the underlying bug. Fix the injection *and* set the header.

## References

- OWASP Secure Headers Project
- OWASP Content Security Policy Cheat Sheet
- MDN — HTTP headers (CSP, Strict-Transport-Security)
