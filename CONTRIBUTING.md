# Contributing

## Rules

One skill per pull request. Reviewing ten skills in one diff means none of them get read properly.

Keep the frontmatter valid — the six fields in [docs/skill-format.md](docs/skill-format.md), `name` unique across the repo, `description` written as a when-to-use sentence.

No theory without a command that proves it. If you can't show the thing running, it isn't a skill yet.

Offensive content ends with the fix. See the offensive/defensive rule in the format doc.

## Adding a skill

1. Pick the domain folder under `skills/`.
2. Create `NN-your-skill-name/SKILL.md`, numbered after the last existing skill in that domain.
3. Fill the sections that apply, drop the ones that don't.
4. Add the row to that domain's `README.md` table and remove the matching `TODO` if there was one.
5. Open the PR against `main`.

## Style

Write like you're explaining it to another engineer who's competent but hasn't seen this particular thing. No marketing, no filler transitions. Commands and output belong in fenced blocks. If a sentence doesn't help the reader do the task, cut it.

## What gets rejected

- Payloads targeting current unpatched software with no defensive framing.
- Malware source or techniques whose only use is evading defenders.
- Anything aimed at systems you don't own.
- A wall of links with no procedure.
- A skill copied from a tool's README with nothing added.
