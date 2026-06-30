---
name: make-git-commit
description: |
  Use this skill when the user wants to create a Git commit following the
  Conventional Commits specification.
---

## Staging

Trust staged state.
Commit straight from staged state, leaving edit, lint, test, and build aside.
Stage files explicitly with paths user named.

## Type

Respect Conventional Commits standard.
Pick type from standard set.
Hold to that standard set.

## Description

Write description in imperative mood, lowercase.
Use no trailing period.
Keep description under 72 characters.
Prepend ticket number with `#` to description.
Do this when commit ties to known issue or pull request.

## Breaking

Append `!` after type or scope when commit introduces incompatible change.
Add `BREAKING CHANGE:` footer when commit introduces incompatible change.

## Body

Keep body to few short sentences.
Explain motivation and visible effect.
Omit body when subject says enough.
Keep message free of restated diffs, file lists, and obvious summaries.
Keep each commit to one related change.

## Footers

Add `Closes #<number>` or `Fixes #<number>` footer only when needed.
Add it only when host needs that token to auto-close ticket on merge.

## Trailers

Keep every `Co-authored-by:` trailer free of Claude, Anthropic, and any agent.
Leave author and committer identity as Git sets them.
Keep message free of advertising trailers, signatures, and emoji.
Keep message free of `Generated with` lines.

## Safety

Run hooks and signing as configured unless user asked to bypass them.
Confine work to one fresh local commit.
Leave amend, rebase, force-push, and push aside.

## Verify

Verify commit subject, body, footers, and file set after committing.
Create new commit on top rather than amending when something is wrong.

## Example

```text
Input: staged src/parser.rs; ties to issue #214; user says "commit it"
git commit produces:
  fix: #214 reject trailing comma in arrays

  The array parser accepted a trailing comma, diverging from the
  JSON grammar. Tighten the lookahead so a stray comma now fails.

  Closes #214

Verified description leads with #214, stays under 72 chars,
lowercase, no trailing period; footer auto-closes the issue.
```
