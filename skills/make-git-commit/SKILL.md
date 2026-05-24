---
name: make-git-commit
description: |
  Use this skill to create a Git commit following the
  Conventional Commits 1.0.0 specification.
---

Trust the staged state, and do not edit files, lint, test, or build before committing.
Stage files explicitly with the paths the user named.
Respect the Conventional Commits 1.0.0 standard.
Pick the type from the standard set, and never invent new ones.
Write the description in imperative mood, lowercase, with no trailing period, under 72 characters.
Prepend the ticket number with a `#` to the description when the commit is tied to a known issue or pull request.
Append `!` after the type or scope and add a `BREAKING CHANGE:` footer when the commit introduces an incompatible change.
Keep the body to a few short sentences explaining motivation and visible effect, or omit it when the subject says enough.
Do not pad the message with restated diffs, file lists, or obvious summaries.
Do not bundle unrelated changes into one commit.
Add a `Closes #<number>` or `Fixes #<number>` footer only when the host needs that token to auto-close the ticket on merge.
Never add a `Co-authored-by:` trailer naming Claude, Anthropic, or any other coding agent.
Never override the author or committer identity.
Never add promotional trailers, signatures, emoji, or `Generated with` lines.
Do not bypass hooks or signing unless the user asked.
Do not amend, rebase, force-push, or push to any remote.
Verify the commit's subject, body, footers, and file set after committing.
Create a new commit on top rather than amending when something is wrong.
