---
name: metadata-service-attacks
domain: 06-cloud-security
description: Use when assessing whether SSRF or a foothold can reach a cloud instance metadata service to steal credentials — and how IMDSv2 and network controls stop it.
difficulty: intermediate
tags: [cloud, aws, ssrf, imds, credential-theft]
tools: [curl]
---

## Purpose

Every cloud VM has a metadata endpoint at `169.254.169.254` that, among other things, hands out the instance's IAM credentials. If an attacker can make a request reach it — usually via SSRF — they get temporary cloud credentials with whatever the instance role allows. This skill covers testing for that reachability and locking the endpoint down.

## When to use it

Whenever you find SSRF (see the web-app SSRF skill) or land a foothold on a cloud instance and want to know what it can reach. Also as a proactive hardening check on your own fleet — is IMDSv2 enforced everywhere?

## Procedure

1. From the instance (or through an SSRF that reaches it), check whether the legacy **IMDSv1** answers a plain GET — no token, no headers. If it does, credential theft is one request away:
   ```
   curl http://169.254.169.254/latest/meta-data/
   ```
2. Walk to the IAM credentials path and read the role name, then the credentials themselves:
   ```
   curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
   curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
   ```
   A JSON blob with `AccessKeyId`, `SecretAccessKey`, and `Token` is live instance credentials.
3. Confirm whether **IMDSv2** is enforced. v2 requires a PUT to get a session token first, and that token must ride on the actual data request — a step SSRF usually can't perform:
   ```
   TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 60")
   curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/
   ```
   If step 1 failed but this works, v2 is enforced — good.
4. For the other providers, the endpoints differ and GCP/Azure already require a header, which blunts naive SSRF:
   ```
   # GCP (needs Metadata-Flavor header)
   curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
   # Azure (needs Metadata:true header + resource param)
   curl -H "Metadata:true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"
   ```
5. If you obtain credentials in an authorised test, verify their power with `sts get-caller-identity` and stop there — enumerate the role's reach, don't act destructively.

## Cheatsheet

```bash
# IMDSv1 (vulnerable to bare SSRF)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# IMDSv2 (token required — usually blocks SSRF)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/

# verify stolen creds (authorised tests only)
aws sts get-caller-identity
```

## Reading the output

- **IMDSv1 returning credentials to a plain GET** = critical. Any SSRF on that instance is cloud-credential theft.
- **The `iam/security-credentials/<role>` JSON** is live, temporary credentials — treat exposure as an active incident, not a finding to schedule.
- **v2 enforced (PUT-for-token required)** significantly reduces SSRF risk, because most SSRF primitives can't send the PUT and custom header. Note it as a mitigating control.
- **A powerful instance role** (`get-caller-identity` plus broad permissions) turns a modest SSRF into account-level compromise — rate the finding by the role's reach.

## The fix

- **Enforce IMDSv2** and disable v1. On AWS, set the instance metadata options to `http-tokens: required`; enforce it org-wide so new instances can't ship with v1:
  ```
  aws ec2 modify-instance-metadata-options --instance-id i-... --http-tokens required --http-endpoint enabled
  ```
- **Lower the hop limit** (`http-put-response-hop-limit: 1`) so a containerised SSRF can't reach the endpoint through an extra network hop.
- **Least-privilege instance roles.** The metadata endpoint is only as dangerous as the role behind it — scope it tightly so stolen credentials do little.
- **Fix the SSRF too.** IMDSv2 is defence in depth; the application-layer SSRF that reaches the endpoint is still the root cause (see the SSRF skill).
- Where an instance doesn't need a role at all, don't attach one.

## Pitfalls

- **Assuming IMDSv2 alone is enough.** It blocks most SSRF, not a genuine foothold on the box — a shell can complete the token PUT. Keep the role least-privilege.
- **Enforcing v2 without checking app compatibility.** Old SDKs and scripts that use v1 will break; update them first, then enforce.
- **Ignoring the hop limit.** Default hop limits can let container/proxy SSRF reach the endpoint even under v2 in some setups.
- **Under-rating the finding.** "SSRF, low" ignores that it may equal cloud admin depending on the instance role. Check the role.

## References

- AWS — Use IMDSv2 documentation
- OWASP SSRF Prevention Cheat Sheet
- GCP and Azure instance metadata documentation
- CWE-441, CWE-918
