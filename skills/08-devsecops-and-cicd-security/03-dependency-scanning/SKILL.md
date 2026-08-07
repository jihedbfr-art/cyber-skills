---
name: dependency-scanning
domain: 08-devsecops-and-cicd-security
description: Use when scanning project dependencies for known vulnerabilities in CI — catching vulnerable third-party packages before they ship, and telling exploitable from merely-flagged.
difficulty: beginner
tags: [devsecops, dependencies, sca, cicd, vulnerabilities]
tools: [dependabot, snyk, trivy, osv-scanner]
---

## Purpose

Most of an application is third-party code, and vulnerable dependencies are one of the most common ways applications get compromised (Log4Shell being the famous case). Software Composition Analysis / dependency scanning checks your dependencies against vulnerability databases and flags known-vulnerable packages. This skill covers integrating dependency scanning into CI and — the harder part — turning its output into action without drowning in unexploitable findings.

## When to use it

A core CI security control for any project with dependencies (nearly all). It's high-value and low-effort to add, but like SAST, its worth depends on managing the output so real risks get fixed and noise doesn't cause fatigue.

## Procedure

1. **Scan dependencies in CI on every build (and on a schedule).** Wire a scanner (Dependabot, Snyk, Trivy, OSV-Scanner) into the pipeline to check declared and transitive dependencies against vulnerability databases. Scan on commits *and* on a schedule — new vulnerabilities are disclosed against dependencies you haven't changed (the vuln-mgmt "scan continuously" principle).
2. **Cover transitive dependencies, not just direct ones.** Most dependencies are transitive (dependencies of your dependencies), and vulnerabilities there count just as much. Ensure the scanner reads the full dependency tree (lockfiles), not only your direct declarations.
3. **Distinguish exploitable from merely-flagged — the key discipline.** A scanner flags every dependency with a known CVE, but many aren't exploitable in your context (the vulnerable function is never called, the vulnerable code path is unreachable). Prioritise by reachability and exploitability (reachability analysis, where available, and the EPSS/KEV signals from vuln-mgmt) so effort goes to real risk, not every flagged package.
4. **Prioritise fixes** — a vulnerable, internet-facing, actually-called dependency with a known exploit is urgent; an unreachable low-severity one can wait. Apply the vuln-mgmt prioritisation (cvss-in-context, EPSS, KEV) to the dependency findings.
5. **Automate updates where safe.** Tools like Dependabot can open PRs to bump vulnerable dependencies automatically; combined with good tests, this keeps dependencies current with low effort. Automated dependency PRs are one of the highest-leverage DevSecOps practices.
6. **Gate builds carefully.** Fail builds on high-severity, exploitable, fixable vulnerabilities (there's a patched version to move to); report the rest without blocking. Blocking on every flagged CVE (including unexploitable ones) causes the same fatigue as noisy SAST.
7. **Generate an SBOM** so you can answer "are we affected by CVE-X?" instantly when the next big vulnerability drops (the supply-chain domain).

## Cheatsheet

```
most of your app = third-party code ; vulnerable deps = a top compromise vector (Log4Shell)
  SCA/dependency scanning = check deps vs vuln DBs

integrate
  SCAN in CI on commits + on a SCHEDULE (new CVEs against unchanged deps)
  cover TRANSITIVE deps (deps of deps — most of them) via lockfiles, not just direct

the key discipline: EXPLOITABLE vs merely-FLAGGED
  scanner flags every dep with a CVE ; many not exploitable in YOUR context
  (vulnerable function never called / path unreachable)
  -> prioritise by reachability + EPSS/KEV (vuln-mgmt), not every flagged package

prioritise fixes (cvss-in-context + EPSS + KEV) ; AUTOMATE updates (Dependabot PRs) — high leverage
GATE on high-severity + exploitable + FIXABLE ; report rest (block-everything -> fatigue)
generate SBOM -> answer "affected by CVE-X?" instantly (supply-chain)
```

## Reading the results

- **A vulnerable, reachable, internet-facing dependency with a known exploit** = the actionable priority; a real compromise vector. This is what dependency scanning should surface to the top.
- **A flagged CVE in a dependency whose vulnerable function you never call** = often not exploitable in your context; treating every flagged package as urgent wastes effort and causes fatigue. Distinguish exploitable from merely-flagged — the core discipline.
- **Transitive dependency vulnerabilities** = count as much as direct ones and are easy to miss if the scanner only reads direct declarations; ensure full-tree coverage.
- **A fixable high-severity vulnerability** = there's a patched version; these are the clear gate-and-fix candidates. Automated PRs (Dependabot) handle many of them.
- **A one-time scan** = stale fast; new CVEs land against unchanged dependencies constantly. Scan on a schedule, not just on change.
- **Prioritised, exploitability-filtered findings with automated updates and an SBOM** = dependency scanning working; real risks fixed fast, noise filtered, and instant answers when the next big CVE drops.

## Pitfalls

- **Treating every flagged CVE as urgent.** Many aren't exploitable in your context; the key discipline is distinguishing exploitable from merely-flagged, or you drown in noise and fatigue sets in.
- **Missing transitive dependencies.** Most vulnerabilities are in dependencies-of-dependencies; scanning only direct declarations misses them. Read the full lockfile tree.
- **One-time scanning.** New CVEs are disclosed against unchanged dependencies daily; a scan goes stale fast. Scan continuously.
- **Over-gating.** Blocking builds on every flagged CVE (including unexploitable ones) causes the same fatigue as noisy SAST. Gate on exploitable, fixable, high-severity.
- **Not automating updates.** Manual dependency bumps lag; automated PRs (with good tests) are one of the highest-leverage practices. Use them.
- **No SBOM.** Without it, "are we affected by the new CVE?" is a frantic manual hunt; generate one.

## References

- Dependabot, Snyk, Trivy, OSV-Scanner documentation
- The vulnerability-management domain (cvss-in-context, EPSS, KEV) and software-supply-chain domain (SBOM)
- OWASP Dependency-Check and SCA guidance
- The sast-integration and build-provenance-slsa skills
