---
name: injection-patterns
domain: 10-secure-code-review
description: Use when reviewing code for the specific shapes that turn into SQLi, command, LDAP, or template injection — the concatenation and interpolation patterns worth stopping on.
difficulty: intermediate
tags: [code-review, injection, sql, command, patterns]
tools: [ripgrep, semgrep]
---

## Purpose

Taint tracking tells you *whether* input reaches a sink. This skill is about recognising the sink shapes fast — the handful of concrete code patterns that become injection when tainted data flows in. Once you can spot them on sight, review gets a lot quicker: you stop on the risky line instead of reading every line with equal suspicion.

## When to use it

Any injection-class review, and especially fast PR review where you don't have time to trace every variable. You scan for the shape first, then taint-track only the ones that light up.

## The shapes that matter

The common denominator is **building a command in a string, then handing the string to an interpreter**. Whenever you see structure (a query, a shell line, a path, a template) assembled by concatenation or interpolation with a runtime value, slow down.

- **SQL** — string-built queries. `"SELECT ... WHERE id = " + id`, f-strings in `cursor.execute`, `String.format` into a query, Hibernate/JPA `createQuery("... where name = '" + name + "'")`. The safe shape is a parameter placeholder (`?`, `:name`) with the value passed separately.
- **Command / OS** — `Runtime.exec("sh -c " + cmd)`, `os.system`, `subprocess` with `shell=True`, `ProcessBuilder` fed a joined string. Safe shape: argument array, no shell.
- **LDAP** — filters built by concatenation: `"(uid=" + user + ")"`. A `)` or `*` in the input rewrites the filter.
- **Template / SSTI** — user data compiled *as* a template rather than passed *to* one: `render_template_string(userInput)`, Thymeleaf/Freemarker fed a user-controlled template name or body.
- **NoSQL** — Mongo query objects assembled from raw request bodies (`{ $where: req.body.q }`), or operators leaking in because the value wasn't cast to a string.

I lost an afternoon once to a JPA `@Query` that looked parameterised — it had `:status` bound properly — but a second clause a few lines down concatenated an "internal only" sort column that turned out to come from the request. The parameterised part lulled me. Read the whole statement, not the first binding.

## Procedure

1. Grep the sink shapes for the stack you're in (cheatsheet below).
2. For each hit, decide in one glance: is the dangerous part a **placeholder** (safe by construction) or **concatenation/interpolation** (guilty until cleared)?
3. Only for the concatenation cases, taint-track the value back to its source. Placeholder-based ones you can usually pass over.
4. For anything dynamic that genuinely can't be parameterised (a table or column name — those can't be bound), confirm it's checked against an **allowlist**, not escaped. Escaping identifiers is a losing game.

## Cheatsheet

```bash
# SQL built by concatenation/format, not binding
rg -n '"(SELECT|INSERT|UPDATE|DELETE).*"\s*\+|execute\(f"|String\.format\(.*(SELECT|WHERE)'
# shell out
rg -n 'shell=True|Runtime\.getRuntime|ProcessBuilder|os\.system|`.*\$'
# LDAP / template / nosql
rg -n '\(uid=|\(cn=|render_template_string|\$where|createQuery\('
```

## The tell vs the false alarm

- A query with `?`/`:named` placeholders and values passed as args = safe shape; don't flag.
- The same query with `+ variable` in the middle = the shape you're hunting.
- `subprocess.run(["ls", path])` (array, no shell) = safe; `subprocess.run("ls " + path, shell=True)` = flag.
- Dynamic **identifiers** (sort column, table) can't be parameterised — the only safe form is allowlist mapping (`{"date": "created_at"}.get(sortKey)`), so if you see escaping there, it's a finding.

## The fix, briefly

Parameterise (SQL), pass argument arrays without a shell (command), bind or allowlist (LDAP, identifiers), never compile user input as a template. The per-class detail lives in domain 03 (web) and 04 (api); this skill is about catching the shape during review so it never ships.

## Pitfalls

- **Trusting the first binding.** A statement can be half-parameterised and half-concatenated. Read all of it.
- **Treating an ORM as automatically safe.** ORMs expose raw-query and `String`-filter escape hatches; those are back to manual injection.
- **Escaping identifiers.** Column/table names aren't bindable and aren't safely escapable — allowlist them.

## References

- OWASP Injection Prevention Cheat Sheet
- CWE-89 (SQL), CWE-78 (OS command), CWE-90 (LDAP), CWE-1336 (template)
