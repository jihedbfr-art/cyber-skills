---
name: anomaly-baselining
domain: 20-threat-hunting
description: Use when hunting requires knowing what normal looks like — establishing baselines of normal behaviour so anomalies stand out, the foundation most hunts depend on.
difficulty: intermediate
tags: [threat-hunting, baselining, anomaly, normal, foundation]
tools: []
---

## Purpose

You can't spot abnormal without knowing normal — and most threat hunts are, at their core, anomaly detection. Attackers do things that deviate from an environment's normal patterns, but "normal" varies enormously between organisations, so a generic idea of suspicious isn't enough. Anomaly baselining is establishing what normal looks like *for your environment* so the anomalies that matter surface. This skill covers building and using baselines, the quiet foundation under lateral-movement, LOTL, beaconing, and most other hunts.

## When to use it

Underpinning most hunts — nearly every hunt in this domain assumes you can distinguish normal from anomalous. Build baselines before or alongside hunting; a hunt without a baseline either misses subtle anomalies or drowns in false ones.

## Procedure

1. **Baseline the dimensions your hunts need.** Different hunts need different baselines: authentication (who logs into what, from where, when), network (what talks to what, normal volumes and destinations), process/execution (what runs normally on which hosts), and user behaviour (normal working hours, access patterns). Baseline what the hunt will compare against.
2. **Establish normal over a representative period.** A baseline needs enough time to capture the real range of normal — including periodic patterns (backups at night, month-end processing, weekly jobs). Too short a window mistakes normal-but-infrequent activity for anomalous.
3. **Account for legitimate variation.** Normal isn't a single value — it's a range with legitimate variation (admins do log into servers; some users travel). A good baseline captures the distribution, not just an average, so genuine anomalies stand out from normal variance.
4. **Use the baseline to surface deviations.** Once you know normal, hunt the deviations: the account logging in at 3am when it never does, the host talking to a destination nothing else touches, the process that's never run on that system before. The deviation *from your baseline* is the lead.
5. **Segment baselines by context.** Normal for a domain controller differs from normal for a workstation; normal for an admin differs from a standard user. Baseline per relevant group so an anomaly is measured against the right normal, not a meaningless global average.
6. **Keep baselines current.** Environments change — new systems, new applications, changed work patterns. A stale baseline flags legitimate change as anomalous (false positives) and may hide new normal that's actually malicious. Refresh baselines as the environment evolves.
7. **Recognise the limits.** Anomaly ≠ malicious — most anomalies are benign (a new legitimate app, an unusual but authorised action). Baselining surfaces *candidates* for investigation, not conclusions; the hunt still has to determine whether an anomaly is actually a threat.

## Cheatsheet

```
can't spot abnormal without knowing NORMAL. most hunts = anomaly detection.
  "normal" varies hugely per org -> generic-suspicious isn't enough -> baseline YOURS

baseline the dimensions your hunts need
  AUTH (who->what, from where, when) | NETWORK (what talks to what, volumes, dests)
  | PROCESS (what runs where) | USER (hours, access patterns)

build it right
  representative PERIOD (capture periodic patterns: nightly backups, month-end)
  capture the RANGE/distribution, not just an average (normal has legit variation)
  SEGMENT by context (DC vs workstation ; admin vs user — right normal, not global avg)

use it: hunt DEVIATIONS from your baseline (3am logon that never happens, new dest, never-run process)
keep CURRENT (stale -> flags legit change as anomaly, hides new malicious normal)

LIMIT: anomaly != malicious. surfaces CANDIDATES, not conclusions. hunt confirms.
```

## Reading with a baseline

- **A deviation from your established baseline** (an account active at a time it never is, a host reaching a destination nothing else does, a process never seen on that system) = the lead most hunts are built to find; the deviation *from your normal* is what matters, not a generic notion of suspicious.
- **No baseline** = you either miss subtle anomalies (they don't look suspicious in isolation) or flag everything unusual (drowning in benign anomalies). The baseline is what makes the hunt tractable.
- **An anomaly measured against a global average** (rather than the right per-group normal) = misleading; a server behaving like a server looks anomalous against a workstation baseline. Segment by context.
- **A stale baseline flagging a new legitimate app/system** = false positives from environment change; refresh baselines as things evolve, or you'll chase legitimate change and lose trust.
- **An anomaly that turns out benign** (a new authorised tool, an unusual but legitimate action) = expected; most anomalies are benign. The baseline surfaces candidates; the hunt determines which are real threats.
- **Context-segmented, current baselines surfacing real deviations** = the foundation that makes anomaly-based hunts work.

## Pitfalls

- **Hunting without a baseline.** Most hunts are anomaly detection; without knowing your normal you miss subtle anomalies or drown in false ones. Baseline first.
- **Too-short a baseline period.** It misses periodic legitimate activity (nightly/monthly jobs), flagging normal-but-infrequent as anomalous. Use a representative window.
- **Baselining an average, not a range.** Normal has legitimate variation; a single-value baseline flags normal variance as anomalous. Capture the distribution.
- **Global instead of segmented baselines.** Measuring a DC against workstation-normal (or vice versa) produces meaningless anomalies. Segment by context.
- **Stale baselines.** Environments change; an outdated baseline flags legitimate change and may accept new malicious activity as normal. Refresh it.
- **Treating anomaly as malicious.** Most anomalies are benign; baselining produces investigation candidates, not verdicts. The hunt still has to confirm.

## References

- The lateral-movement-hunting, living-off-the-land, beaconing-detection, and dns-and-proxy-hunting skills (all depend on baselining)
- The detection reducing-false-positives skill (baselining known-good) and SOC enrichment-and-context skill
- SANS threat hunting and UEBA (user/entity behaviour analytics) resources
