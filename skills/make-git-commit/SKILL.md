---
name: make-git-commit
description: |
  Use this skill when the user wants to create a Git commit following the
  Conventional Commits 1.0.0 specification.
---

## Staging

Trust staged state.
Do not edit files, lint, test, or build before committing.
Stage files explicitly with paths user named.

## Type

Respect Conventional Commits 1.0.0 standard.
Pick type from standard set.
Never invent new ones.

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
Do not pad message with restated diffs, file lists, or obvious summaries.
Do not bundle unrelated changes into one commit.

## Footers

Add `Closes #<number>` or `Fixes #<number>` footer only when needed.
Add it only when host needs that token to auto-close ticket on merge.

## Trailers

Never add `Co-authored-by:` trailer naming Claude, Anthropic, or any agent.
Never override author or committer identity.
Never add promotional trailers, signatures, emoji, or `Generated with` lines.

## Safety

Do not bypass hooks or signing unless user asked.
Do not amend, rebase, force-push, or push to any remote.

## Verify

Verify commit subject, body, footers, and file set after committing.
Create new commit on top rather than amending when something is wrong.
