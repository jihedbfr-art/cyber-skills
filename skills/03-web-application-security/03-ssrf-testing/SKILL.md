---
name: ssrf-testing
domain: 03-web-application-security
description: Use when an app fetches a URL you can influence — testing whether you can make the server request internal services, cloud metadata, or arbitrary hosts, and the fix.
difficulty: intermediate
tags: [owasp, ssrf, web, cloud, injection]
tools: [burp, curl]
---

## Purpose

Server-Side Request Forgery is making the server fetch a URL of your choosing. Because the request comes *from the server*, it reaches things you can't: internal admin panels, databases, and — the big one in cloud — the metadata service that hands out credentials. This skill covers finding SSRF and shutting it down.

## When to use it

Any feature where the server fetches something on your behalf: URL preview/unfurling, webhook configuration, "import from URL", PDF/screenshot generators, image proxies, XML/SVG parsers, anything taking a URL, hostname, or IP as input.

## Procedure

1. Find the fetch. Look for parameters holding URLs or hostnames, and features that clearly reach out (link previews, avatar-from-URL, webhook tests).
2. Point it at a listener you control and confirm the server actually calls out. Use a request-catcher (Burp Collaborator, or a simple server you own):
   ```
   url=https://your-collaborator-id.oastify.com/ssrf-check
   ```
   A hit means the server fetched it — SSRF is live.
3. Pivot inward. Try to reach the loopback interface and common internal ports:
   ```
   url=http://127.0.0.1:80/
   url=http://localhost:8080/admin
   ```
4. In a cloud environment, test the metadata endpoint — this is where SSRF turns into credential theft. Expect and want a block (see IMDSv2 in the cloud domain):
   ```
   url=http://169.254.169.254/latest/meta-data/
   ```
5. If direct internal URLs are filtered, test the usual bypasses to see whether the filter is real or cosmetic: alternate IP encodings, a domain that resolves to `127.0.0.1`, redirect chains, and non-`http` schemes:
   ```
   url=http://127.1/
   url=http://[::1]/
   url=http://2130706433/            # decimal for 127.0.0.1
   url=http://your-domain-that-a-records-to-127.0.0.1/
   ```
6. Note whether responses come back to you (in-band) or not (blind). Blind SSRF is still serious — the metadata pivot doesn't need the response echoed if the app uses the fetched content.

## Cheatsheet

```
# confirm outbound fetch
url=https://COLLAB-ID.oastify.com/x

# internal pivot
http://127.0.0.1:PORT/     http://localhost/     http://[::1]/

# cloud metadata (credential theft target)
http://169.254.169.254/latest/meta-data/iam/security-credentials/   # AWS
http://metadata.google.internal/computeMetadata/v1/                 # GCP
http://169.254.169.254/metadata/instance?api-version=2021-02-01      # Azure

# filter bypasses
http://127.1/    http://2130706433/    http://0x7f000001/
http://localtest.me/    (public name -> 127.0.0.1)
gopher:// file:// dict://   (non-http schemes, if the fetcher allows)
```

## Reading the output

- **A callback on your listener** confirms SSRF, even if nothing comes back in the response — that's blind SSRF and still exploitable.
- **An internal service's response** (a title, a redirect, a JSON body from a port you can't reach directly) proves you're reaching the internal network.
- **Metadata credentials returned** is critical — that's cloud account access, the worst SSRF outcome.
- **A filter that blocks `127.0.0.1` but not `127.1` or the decimal form** is a broken filter — report it as vulnerable, not mitigated.

## The fix

Deny by default. Validate the destination against an **allowlist** of exact hosts the feature legitimately needs, not a blocklist of bad ones (blocklists lose to encoding tricks every time). Then:

- Resolve the hostname and check the *resolved IP* against the allowlist, after following redirects — a name that passes validation can still resolve to `169.254.169.254` or `10.0.0.0/8` (DNS rebinding). Re-validate the final IP.
- Block requests to private, loopback, and link-local ranges (RFC 1918, `127.0.0.0/8`, `169.254.0.0/16`).
- Allow only `http`/`https`; reject `file://`, `gopher://`, `dict://`.
- Don't return the raw fetched response to the user, and give the fetcher its own restricted network egress.
- On AWS specifically, enforce **IMDSv2** so a bare SSRF can't read the metadata endpoint — defence in depth for when the app-layer check fails.

## Pitfalls

- **Blocklist filters.** `127.0.0.1` blocked, `2130706433` allowed — same address, different notation. Allowlist the destination instead.
- **Validating the URL but not the resolved IP.** DNS rebinding walks straight through hostname checks.
- **Dismissing blind SSRF.** No echoed response doesn't mean no impact — metadata theft and internal port scanning don't need the body returned.
- **Forgetting redirects.** An allowed host that 302s to an internal one defeats a naive check. Validate after each hop.

## References

- OWASP WSTG-INPV-19 (Testing for SSRF)
- OWASP SSRF Prevention Cheat Sheet
- CWE-918
- AWS IMDSv2 documentation
