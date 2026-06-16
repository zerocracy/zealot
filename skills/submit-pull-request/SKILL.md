---
name: submit-pull-request
description: |
  Use this skill when the user wants to open a single pull
  request against a GitHub repository.
---

## Scope

Target GitHub repository user named.
Operate from feature branch that master skill prepared in local working tree.
Identify base branch from repository's actual default.
Do not hard-code `main` or `master`.

## Rebase

Fetch and rebase branch onto latest base before opening pull request.
Resolve every rebase conflict by hand.
Re-run build after each conflict resolution.
Never use `git rebase --skip` or `-X ours` and `-X theirs` shortcuts.

## Build

Run full build before pushing branch.
Run every test, every linter, and every static check named in `README.md`.
Run every check named in `CLAUDE.md` too.
Stabilize build before pushing branch.
Push branch.
Confirm push succeeded before opening pull request.

## Title

Compose title as short, declarative sentence in imperative mood.
Keep title lowercase except for proper nouns.
Keep title under 72 characters with no trailing punctuation.
Prepend ticket number with `#` when pull request closes or refers to issue.
Match title to dominant commit type when project's commit style expects it.

## Body

Write body as three short paragraphs.
First paragraph covers motivation and linked issue.
Second paragraph covers visible effect of change.
Third paragraph covers checks you ran with their outcome.
Add `Closes #<number>` or `Fixes #<number>` line on its own.
Add that line when pull request resolves open issue.
Ground test-plan paragraph in commands that actually ran on local machine.
Keep body compact.

## Options

Set base and head branches explicitly even when they match defaults.
Mark pull request as draft only when user explicitly requested draft.
Never mark it ready-for-review automatically.
Do not request reviewers unless user explicitly asked.
Do not assign labels, set milestones, or assign pull request unless asked.
Do not merge, approve, auto-merge, or force-push pull request after it opens.

## CI

Capture pull request URL.
Watch CI once so first-build failure surfaces inside run.
Fix CI failures inside this run.
Commit and push additional commits to same branch.
Do not consider pull request submitted until every required check is green.
Stop when user accepts red status instead.
Stop as soon as every required CI check on latest commit is green.

## Report

Do not poll for reviewer comments, reviews, or approvals after CI turns green.
Report short factual summary of pull request URL.
Report base and head branches.
Report commits you pushed during run.
Report final CI status.
