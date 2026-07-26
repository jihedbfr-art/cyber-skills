---
name: s3-bucket-misconfiguration
domain: 06-cloud-security
description: Use when checking AWS S3 (or S3-compatible) buckets for public read/write exposure — the classic cloud data leak — and locking them down.
difficulty: beginner
tags: [cloud, aws, s3, storage, misconfiguration, data-exposure]
tools: [aws-cli, curl]
---

## Purpose

A huge share of "cloud breaches" are just object storage left open. No exploit, no CVE — a bucket set to public, or writable by anyone, holding backups, PII, or credentials. This skill covers checking whether a bucket is exposed, and the settings that close it for good.

## When to use it

Auditing your own AWS account, or an authorised assessment where S3 is in scope. Also whenever you find a bucket name in a page's source, a mobile app, or a subdomain (`assets.example.com` pointing at a bucket) — that's a lead worth checking against your scope.

Only test buckets you own or are authorised to test. Reading a stranger's bucket, even an open one, is unauthorised access.

## Procedure

1. Identify the bucket name. It shows up in URLs (`https://bucket-name.s3.amazonaws.com`), app configs, and CNAMEs.
2. Check anonymous read on the bucket listing. A returned XML key list means public listing is on:
   ```
   curl -s "https://bucket-name.s3.amazonaws.com/"
   ```
3. Check anonymous object read — a bucket can block listing but still serve objects to anyone with the key:
   ```
   curl -s "https://bucket-name.s3.amazonaws.com/known-file.txt"
   ```
4. With credentials for your own account, inspect the actual controls rather than guessing from responses:
   ```
   aws s3api get-bucket-acl --bucket bucket-name
   aws s3api get-public-access-block --bucket bucket-name
   aws s3api get-bucket-policy --bucket bucket-name
   ```
5. Test anonymous *write* — the more dangerous case, since it lets an attacker plant or tamper with content. Expect (and want) an `AccessDenied`:
   ```
   curl -s -X PUT -d "test" "https://bucket-name.s3.amazonaws.com/pentest-write-check.txt"
   ```
6. Record what you found with the exact ACL/policy that caused it — "public" isn't actionable; "the bucket policy allows `s3:GetObject` for `Principal: *`" is.

## Cheatsheet

```bash
# anonymous listing / object read
curl -s "https://BUCKET.s3.amazonaws.com/"
curl -s "https://BUCKET.s3.amazonaws.com/path/file"

# authenticated inspection (your own account)
aws s3api get-bucket-acl --bucket BUCKET
aws s3api get-public-access-block --bucket BUCKET
aws s3api get-bucket-policy --bucket BUCKET
aws s3 ls s3://BUCKET --no-sign-request       # test anonymous list via CLI

# account-wide: find buckets without the public-access block
aws s3api list-buckets --query 'Buckets[].Name' --output text
```

## Reading the output

- **A key listing (XML `<ListBucketResult>` with `<Contents>`)** from an unauthenticated request = public listing. Anyone can inventory the bucket.
- **An object body returned to `--no-sign-request`** = public read, even if listing is off. Attackers guess or find key names.
- **`get-public-access-block` showing `false`/absent** on the four settings = the account-level guardrail is off. This is the single most useful signal.
- **A bucket policy with `"Principal": "*"` and `"Effect": "Allow"`** on `s3:GetObject` (or worse, `s3:PutObject`) = the exposure, in the exact words for your report.
- **`AccessDenied` on the write test** = good, writes are blocked.

## The fix

1. **Turn on Block Public Access**, ideally at the account level so no single bucket can be made public by mistake:
   ```
   aws s3api put-public-access-block --bucket BUCKET \
     --public-access-block-configuration \
     BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
   ```
2. **Remove `Principal: *` grants** from the bucket policy and ACLs. If the public was reading assets, front the bucket with CloudFront and an Origin Access Control instead of exposing the bucket directly.
3. **Prefer IAM policies and pre-signed URLs** over public objects for anything non-static. A pre-signed URL grants time-limited access without making the object public.
4. **Enable default encryption and access logging** so you can encrypt at rest and see who touched what.
5. **Detect drift.** Have Config or a scheduled check flag any bucket where public access gets re-enabled — the fix has to survive the next deploy.

## Pitfalls

- **"It's just static assets."** Public read is defensible for genuinely public content — but confirm the bucket doesn't also hold backups or logs under another prefix. Buckets get reused.
- **Fixing one bucket, ignoring the account.** Without account-level Block Public Access, the next bucket someone creates can repeat the mistake.
- **Reading data to prove exposure.** Confirm access is possible; don't download the contents. For someone else's data that's a real breach, on you.
- **Assuming private = safe.** A bucket private to the internet can still be over-shared internally via an over-broad IAM policy. Check both edges.

## References

- AWS S3 — Blocking public access documentation
- AWS S3 Security Best Practices whitepaper
- OWASP Cloud Security guidance
- CWE-732 (Incorrect Permission Assignment)
