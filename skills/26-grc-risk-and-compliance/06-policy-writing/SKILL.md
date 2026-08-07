---
name: policy-writing
domain: 26-grc-risk-and-compliance
description: Use when writing security policies people actually follow — clear, actionable, enforceable documents that drive behaviour, not shelfware written to satisfy an auditor.
difficulty: beginner
tags: [grc, policy, governance, documentation, compliance]
tools: []
---

## Purpose

Security policies are meant to define required behaviour and controls — but most are written to satisfy an auditor, filled with vague legalese nobody reads, and ignored in practice. A good policy is clear, actionable, and followed. This skill covers writing security policies that actually drive behaviour and hold up, rather than shelfware that ticks a compliance box while changing nothing.

## When to use it

When establishing governance (frameworks like ISO 27001 require documented policies), or when existing policies are ignored/ineffective. Policies are foundational to a security programme — but only if they're usable; an unread policy provides neither security nor real compliance.

## Procedure

1. **Write for the reader, to drive behaviour — the key principle.** A policy exists to make people do (or not do) something; if the intended audience can't understand what's required of them, the policy fails regardless of how compliant it looks. Write in plain language, aimed at the people who must follow it, not at an auditor. Clarity that changes behaviour beats legalese that satisfies a checklist.
2. **Distinguish policy, standard, and procedure.** A **policy** states what's required and why (high-level, stable); a **standard** specifies the how at a detailed level (specific technologies/configs); a **procedure** is step-by-step instructions. Mixing them makes policies bloated and hard to maintain. Keep policies concise and high-level; put the detail in standards/procedures.
3. **Make requirements clear and actionable.** State specifically what people must do, not vague aspirations ("employees shall use strong passwords" → "passwords must be at least X characters and not reused; use the provided password manager"). A reader should know exactly what's expected of them. Ambiguous requirements can't be followed or enforced.
4. **Make it enforceable.** A policy states a requirement; enforcement makes it real. Requirements should be things you can actually check and enforce (technically or through process), with consequences defined. An unenforceable policy is a wish.
5. **Keep it realistic.** A policy people can't reasonably follow gets ignored (and creates workarounds); requirements must be achievable in the real work context. Overly strict, impractical policies breed non-compliance and shadow processes.
6. **Assign ownership and review.** Each policy needs an owner responsible for it and a review cadence — policies go stale as the organisation and threats change. A stale policy misleads and loses credibility. Version and review them.
7. **Communicate and make accessible.** A policy nobody knows about or can't find isn't followed; publish policies where people can find them, communicate changes, and include them in onboarding/awareness. Combine with the reporting/awareness culture so policies are lived, not filed.

## Cheatsheet

```
policies define required behaviour — but most = shelfware (auditor-satisfying legalese, ignored)
  good policy = clear + actionable + FOLLOWED (drives behaviour, not a checkbox)

write it
  FOR THE READER to drive BEHAVIOUR (plain language, aimed at who must follow it, not the auditor)
    -> can't understand = policy fails regardless of how compliant it looks
  POLICY vs STANDARD vs PROCEDURE: policy = what+why (high-level, stable) | standard = detailed how
    | procedure = step-by-step. don't mix -> keep policy concise, detail in standards/procedures
  CLEAR + ACTIONABLE requirements (reader knows exactly what's expected — not vague aspirations)
  ENFORCEABLE (checkable + consequences ; unenforceable = a wish)
  REALISTIC (can't-follow -> ignored + workarounds ; impractical breeds non-compliance + shadow processes)
  OWNER + review cadence (stale policy misleads + loses credibility)
  COMMUNICATE + accessible (unknown/unfindable = not followed ; onboarding + awareness)
```

## Reading a policy

- **Vague legalese nobody reads** = the shelfware failure; a policy that satisfies an auditor but that the intended readers can't understand or act on changes no behaviour. Write in plain language for the reader — clarity that drives behaviour is the point.
- **Requirements that are clear and actionable** ("passwords must be X, use the manager") vs vague aspirations ("use strong passwords") = the difference between a policy people can follow and one they can't. Specificity makes it followable and enforceable.
- **A policy mixing high-level requirements with detailed configs** = bloated and hard to maintain; separate policy (what/why) from standards (detailed how) and procedures (steps).
- **An unenforceable requirement** = a wish, not a policy; requirements should be checkable with defined consequences, or they're ignored.
- **An impractical policy** = ignored, with workarounds and shadow processes; realistic, followable requirements are what actually get complied with.
- **A stale, ownerless policy** = misleads and loses credibility; each policy needs an owner and review cadence.
- **Clear, actionable, enforceable, realistic, owned, communicated policies** = governance that drives behaviour, not shelfware.

## Pitfalls

- **Writing for the auditor, not the reader.** The core failure — a compliant-looking policy the intended readers can't understand or follow changes no behaviour. Write in plain language for who must follow it.
- **Vague requirements.** "Use strong passwords" can't be followed or enforced; state specifically what's required so the reader knows exactly what to do.
- **Mixing policy, standard, and procedure.** It bloats policies and makes them unmaintainable; keep policy high-level, put detail in standards/procedures.
- **Unenforceable or unrealistic policies.** A requirement you can't check, or one people can't reasonably follow, gets ignored and breeds workarounds. Make them enforceable and realistic.
- **No owner or review.** Policies go stale as things change; without ownership and review they mislead and lose credibility.
- **Not communicating them.** A policy nobody knows about or can find isn't followed; publish, communicate, and include in awareness.

## References

- The iso-27001-isms skill (documented policies requirement) and the reporting-culture / awareness skills
- SANS security policy templates and NIST policy guidance
- The engineering documentation skill (clear technical writing)
- ISO 27001/27002 (policy requirements)
