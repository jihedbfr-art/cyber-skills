# Skill format

Every skill in this repo is one folder holding one `SKILL.md`. Same skeleton across the whole library, so a reader — human or agent — always knows where to look.

## Frontmatter

YAML block at the top, between two `---` lines. Six fields, all required except `tools`.

```yaml
---
name: sql-injection-testing
domain: 03-web-application-security
description: Use when checking a web endpoint for SQL injection — covers manual probes, sqlmap, and the fix.
difficulty: intermediate      # beginner | intermediate | advanced
tags: [owasp, injection, database, web]
tools: [sqlmap, burp, curl]
---
```

`name` is kebab-case and unique across the repo. `description` is the important one: it is written as a *when-to-use* sentence because that is the string an agentic assistant matches a task against. A title like "SQL injection" is useless for matching; "Use when checking a web endpoint for SQL injection" is not.

## Body sections

Use the headings that apply, drop the rest. Do not invent content to fill a section.

**## Purpose** — one paragraph. What the skill gets you and what it does not.

**## When to use it** — the concrete situations that should trigger it, and when to reach for something else instead.

**## Procedure** — ordered steps. This is the part an agent executes, so each step is an action, not a concept. Reference the exact command or tool.

**## Cheatsheet** — commands, flags, payloads, and one-liners a human scans without reading the prose. Fenced code blocks.

**## Reading the output** — how to tell a true positive from noise. This is where most tool tutorials stop and where real assessment starts.

**## Pitfalls** — the mistakes that waste an afternoon: false positives, rate limits, things that look broken but are not.

**## References** — primary sources. Official docs, the relevant CWE/CVE, the tool's own manual. No filler links.

## The offensive/defensive rule

An offensive skill ends with the fix. If it shows how to find or exploit a weakness, the last section shows how to close it. A skill that only breaks things does not go in.

## Length

Whatever the skill needs. A tight one is a screen; a broad one runs longer. Padding to hit a size is worse than a short entry that says everything it needs to.
