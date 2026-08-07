# cyber-skills — recap for handoff (Antigravity)

**Repo:** https://github.com/jihedbfr-art/cyber-skills (public, MIT)
**Local:** `F:\cyber-skills`
**Owner/brand:** JihedAiLabs (jihedbfr-art)
**Status as of 2026-08-06:** ✅ **v1 complete — 260 skills, 26/26 domains at 10/10, 268 commits, all pushed.**

---

## What this repo is

A working library of security skills across **26 domains**, built in the viral "cyber-skills / 26 domains"
mould but designed to survive inspection — real procedures, not a link dump.

Each skill is its own folder holding one `SKILL.md` that works **two ways**:
- a **checklist a human scans** (cheatsheet section), and
- a **procedure an agentic coding assistant loads** (YAML frontmatter `name` + trigger-sentence `description`, ordered `Procedure`).

Format spec: `docs/skill-format.md`. Contribution rules: `CONTRIBUTING.md`.

### SKILL.md skeleton
```
---
name, domain, description, difficulty, tags, tools
---
## Purpose
## When to use it
## Procedure        <- ordered steps an agent executes
## Cheatsheet        <- commands/flags a human scans
## Reading the output
## Pitfalls
## References
```

### Repo layout
```
README.md / README.fr.md        bilingual, JihedAiLabs-branded
assets/jihedailabs-logo.svg     brand mark (SVG)
docs/skill-format.md            format spec
CONTRIBUTING.md  LICENSE  .gitignore
skills/NN-domain-name/
   README.md                    domain roster table (all ✅) + brand footer
   NN-skill-name/SKILL.md       260 of these
```

---

## The 26 domains (all 10/10)

| # | Domain | # | Domain |
|---|--------|---|--------|
| 01 | OSINT & Reconnaissance | 14 | Cryptography & PKI |
| 02 | Network Security | 15 | Vulnerability Management |
| 03 | Web Application Security | 16 | Red Teaming & Adversary Emulation |
| 04 | API Security | 17 | Social Engineering Defence |
| 05 | Mobile Security | 18 | Detection Engineering |
| 06 | Cloud Security | 19 | Security Operations & SIEM |
| 07 | Container & Kubernetes Security | 20 | Threat Hunting |
| 08 | DevSecOps & CI/CD Security | 21 | Threat Intelligence |
| 09 | Software Supply Chain Security | 22 | Incident Response |
| 10 | Secure Code Review | 23 | Digital Forensics |
| 11 | Identity & Access Management | 24 | Malware Analysis & Reverse Engineering |
| 12 | Active Directory & Windows Security | 25 | AI & LLM Security |
| 13 | Linux & Unix Security | 26 | GRC, Risk & Compliance |

Each domain = 10 skills. The offensive/defensive split runs throughout: **every offensive skill ends with
the fix**, and the blue-team domains (detection, hunting, SOC, threat-intel, IR, forensics) cross-reference
the offensive ones.

---

## Conventions that shaped it (keep applying)

- **No AI traces:** authored as *Jihed Ben Arfa*, human commit style, no Claude/Anthropic mentions in
  content or history. `.gitignore` blocks `CLAUDE.md` at every depth.
- **Varied voice:** skills deliberately differ in phrasing/structure (humanize discipline) — not one
  template cloned 260×, which would read as mass-generated.
- **Commit granularity:** one commit per skill, message form `domain: skill name skill`.
- **Vendor-neutral:** the "agent skill" format is presented generically; no product named as the origin
  of the idea. A functional model string in example code is fine; naming a product in prose is not.
- **Brand:** JihedAiLabs logo + line on README EN/FR and a footer on each domain roster — **never inside
  SKILL.md** (agent-loaded content stays clean).

---

## Build history (waves)

Built in 24 waves. Waves 1–3 scaffolded all 26 domains + a first anchor skill each; waves 4+ completed
one or more domains per wave (9 skills each) until every domain hit 10/10. Full wave-by-wave log is in
the project memory (`~/.claude/.../memory/project-cyber-skills.md`).

---

## Possible next steps

- **Depth pass:** go beyond 10 skills/domain where a domain warrants it (the roster tables were capped at
  10 by design, not by exhausting the topic).
- **Repo polish:** GitHub topics are set; could add a CI check validating SKILL.md frontmatter, a
  contributor guide expansion, or per-domain "start here" learning paths.
- **Distribution:** package selected domains as an installable agent-skills bundle.
- Or move on — v1 is done and self-contained.

---

*A JihedAiLabs project.*
