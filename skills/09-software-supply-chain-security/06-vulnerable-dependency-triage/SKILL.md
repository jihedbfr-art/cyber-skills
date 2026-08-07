---
name: vulnerable-dependency-triage
domain: 09-software-supply-chain-security
description: Use when triaging flagged vulnerable dependencies — telling exploitable from merely-present so you fix what actually matters instead of chasing every CVE in the dependency tree.
difficulty: intermediate
tags: [supply-chain, dependencies, triage, reachability, prioritisation]
tools: [snyk, osv-scanner]
---

## Purpose

Dependency scanners flag every dependency with a known CVE — and in a real project that's often hundreds of findings, most of which aren't actually exploitable in your context. Chasing all of them wastes effort and causes fatigue; ignoring them risks missing the real one. This skill covers triaging vulnerable dependencies — distinguishing the exploitable from the merely-present — so remediation effort goes where it reduces real risk. It's the dependency-specific application of vulnerability triage.

## When to use it

Whenever dependency scanning (the devsecops skill) produces more findings than you can fix at once — which is always in a mature project. It's the discipline that turns a scanner's raw output into an actionable, prioritised set.

## Procedure

1. **Start from the reality: most flagged CVEs aren't exploitable in your context.** A dependency having a CVE doesn't mean *your* application is vulnerable — the vulnerable function may never be called, the vulnerable code path may be unreachable, or the CVE may require conditions your usage doesn't meet. The core triage question is "is this actually reachable and exploitable here?", not "does this dependency have a CVE?".
2. **Assess reachability — the key discriminator.** Is the vulnerable code actually reachable from your application? Reachability analysis (some scanners offer it) determines whether your code paths actually invoke the vulnerable function. An unreachable vulnerability is far lower priority than a reachable one. This single factor separates most noise from signal.
3. **Assess exploitability in context.** Even if reachable, does exploitation require conditions you don't meet (a specific input the attacker can't control, a configuration you don't use)? And is there a public exploit / is it being exploited (EPSS/KEV from vuln-mgmt)? Combine reachability with real-world exploitability.
4. **Prioritise by the full picture** — reachable + exploitable + exposed (internet-facing) + known-exploited is urgent; unreachable + no exploit + internal is low. Apply the vuln-mgmt prioritisation (cvss-in-context, EPSS, KEV) to the dependency findings.
5. **Remediate the real risks.** For the priorities: update to a fixed version (usually the simplest — automated PRs help), or if no fix exists, mitigate (remove the dependency, avoid the vulnerable path, apply a compensating control) and document.
6. **Handle the no-fix and transitive cases.** A vulnerable transitive dependency you don't directly control may need an override/resolution to force a fixed version, or waiting for the direct dependency to update. A vulnerability with no available fix is a risk-acceptance decision (document it).
7. **Suppress the confirmed non-issues with reasons.** An unreachable or non-applicable finding, once confirmed, should be suppressed with a recorded justification so it doesn't reappear every scan and cause fatigue — but review suppressions periodically (reachability can change with code changes).

## Cheatsheet

```
scanners flag EVERY dep with a CVE -> hundreds of findings, most NOT exploitable in your context
  chase all = wasted effort + fatigue ; ignore all = miss the real one. TRIAGE.

core question: "reachable + exploitable HERE?" — not "does this dep have a CVE?"

1. REACHABILITY (key discriminator): is the vulnerable code actually invoked by your paths?
     reachability analysis -> unreachable vuln = far lower priority. separates most noise from signal.
2. EXPLOITABILITY in context: needs conditions you don't meet? public exploit? EPSS/KEV?
3. PRIORITISE full picture: reachable + exploitable + exposed + known-exploited = urgent
     unreachable + no exploit + internal = low  (cvss-in-context + EPSS + KEV)
4. REMEDIATE priorities: update to fixed version (automated PRs) ; no fix -> mitigate + document
5. TRANSITIVE / no-fix: override/resolution to force fixed version ; no-fix = risk acceptance (documented)
6. SUPPRESS confirmed non-issues WITH reason (don't reappear) — but re-review (reachability changes)
```

## Reading the findings

- **A reachable, exploitable vulnerability in an internet-facing dependency with a known exploit** = the actionable priority; a real risk. This is what triage should surface to the top from the hundreds of findings.
- **A flagged CVE in a dependency whose vulnerable function you never call** = usually not exploitable in your context; the reachability question is what tells you this. Treating it as urgent wastes effort — the core triage discipline is distinguishing this from real risk.
- **Reachability analysis showing a vulnerable path is unreachable** = strong evidence to de-prioritise; the single most useful signal for cutting dependency-scanning noise.
- **A vulnerable transitive dependency** = you don't control it directly; force a fixed version via override/resolution, or wait for the direct dependency to update. A common triage complication.
- **A vulnerability with no available fix** = a risk-acceptance/mitigation decision (remove the dependency, avoid the path, compensating control), documented — not an indefinite open finding ignored silently.
- **Confirmed non-issues suppressed with reasons, priorities remediated** = triage working; effort on real risk, noise filtered, decisions recorded.

## Pitfalls

- **Treating every flagged CVE as a vulnerability in your app.** A dependency's CVE doesn't mean you're exploitable; reachability and context usually make most findings low or non-issues. Triage by "exploitable here", not "has a CVE".
- **Ignoring reachability.** It's the key discriminator between noise and signal; without it you can't tell the exploitable few from the flagged many. Use reachability analysis where available.
- **Chasing all findings equally.** Fixing hundreds of unexploitable CVEs wastes effort and causes fatigue, and the real one gets lost. Prioritise by reachability + exploitability + exposure.
- **Missing transitive vulnerabilities' remediation path.** You can't just "update" a transitive dependency you don't declare; use overrides/resolutions or wait for the direct dependency.
- **Silent suppression or silent ignoring.** Confirmed non-issues should be suppressed *with a reason* (and re-reviewed, since reachability changes); real no-fix risks should be documented acceptances, not quietly ignored.

## References

- The vulnerability-management domain (cvss-in-context, epss, triage-and-deduplication)
- The devsecops dependency-scanning skill and reachability-analysis tooling (Snyk, endor, etc.)
- OSV, GitHub Advisory Database (vulnerability data)
- The lockfile-integrity and malicious-package-response skills
