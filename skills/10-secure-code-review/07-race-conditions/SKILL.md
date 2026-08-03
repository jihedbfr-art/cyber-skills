---
name: race-conditions
domain: 10-secure-code-review
description: Use when reviewing code for TOCTOU and concurrency bugs with a security impact — the double-spend, the check that goes stale before the use, the shared state two requests corrupt.
difficulty: advanced
tags: [code-review, concurrency, toctou, race-condition]
tools: [grep, ripgrep]
---

## Purpose

Race conditions are the review findings that don't reproduce when you read the code once, top to bottom — because the bug only exists when two things happen at the same time. The security-relevant subset is small and recognisable: a security decision made against state that can change before it's acted on, or shared state mutated without protection. This skill is about reading for the interleaving, not the single path.

## When to use it

Reviewing anything that (a) checks a condition then acts on it as two separate steps, (b) mutates state shared across requests/threads/instances, or (c) enforces a limit — balance, quota, one-time use, rate cap. Payment, redemption, and provisioning code deserve this pass every time.

## The two shapes

**TOCTOU — time-of-check to time-of-use.** The code verifies something, then uses it, and the two moments aren't atomic. Between them, the world changes.

- Check "balance >= amount", then debit — two concurrent requests both pass the check, both debit. Double-spend.
- Verify a file's path/permissions, then open it — the path gets swapped (symlink) in between.
- "Is this coupon unused?" then "mark it used" — two requests redeem the same one.
- Check a user exists / is under quota, then create — both create.

**Unsynchronised shared state.** Two requests mutate the same object, static field, cache entry, or file without a lock, and one clobbers the other or leaves it half-updated. In web code this often hides in a static/singleton holding per-request data — which also causes data to leak *between* users, a confidentiality bug, not just a corruption one.

## Procedure

1. Find the check-then-act pairs and the limit enforcement (cheatsheet). For each, ask: **could a second request slip between the check and the act?** If the two steps aren't one atomic operation, assume yes.
2. Find shared mutable state — statics, singletons, module globals, shared caches, instance fields on a shared object. Ask whether concurrent requests can reach it and whether anything serialises them.
3. For each risky spot, decide if the impact is a *security* one: money, access, one-time tokens, quota bypass, or cross-user data bleed. Correctness-only races still matter but this domain cares about the security subset.
4. Check the guard, if any, is real: an application-level `if (!inUse)` is not atomic; a DB unique constraint, `SELECT ... FOR UPDATE`, atomic compare-and-set, or a conditional `UPDATE ... WHERE status='unused'` is.

## Cheatsheet

```bash
# check-then-act around money/limits/one-time use
rg -n 'balance|quota|remaining|isUsed|used ?= ?false|if .*count *<|hasRedeemed'
# shared mutable state
rg -n 'static .*=|@Component.*\n.*private (?!final)|global |threading\.|ThreadLocal'
# the atomic tools you WANT to see instead
rg -n 'FOR UPDATE|SELECT .* LOCK|compareAndSet|synchronized|@Transactional|UNIQUE|ON CONFLICT'
```

## Reading it

- **Non-atomic check-then-act on a limited resource** → the classic exploitable race (double-spend, double-redeem). High value if money or access is involved.
- **A DB unique constraint or `UPDATE ... WHERE status='unused'` returning affected-rows** → correct; the database serialises it. Don't flag.
- **`synchronized`/lock on a single instance in a multi-instance deployment** → false safety; the lock doesn't span JVMs/pods. The guarantee has to live where all requests converge (the DB, a distributed lock).
- **Per-request data on a static/singleton field** → both a race and a cross-user leak. Call out the confidentiality angle.

## The fix

Make the check and the act one atomic operation at the layer that actually serialises: a conditional update that both tests and sets in a single statement, a unique constraint that makes the second attempt fail, `SELECT ... FOR UPDATE` inside a transaction, or an atomic CAS. Push the invariant down to the database or a shared lock — not the application layer, which can't serialise across instances. For shared state, don't share mutable per-request data at all; keep it request-scoped.

## Pitfalls

- **Testing once and calling it safe.** These pass every single-threaded read. You have to reason about two requests.
- **In-process locks in a scaled-out service.** `synchronized`/mutex protect one process; your prod runs many.
- **Trusting an app-level flag.** `if (!used) { use(); }` is two operations. The check and set must be atomic and colocated with the data.
- **Ignoring the leak angle.** Shared per-request state doesn't just corrupt — it can serve one user's data to another.

## References

- CWE-362 (Race Condition), CWE-367 (TOCTOU), CWE-488 (Data Leak Between Sessions)
- OWASP guidance on business-logic and concurrency flaws
