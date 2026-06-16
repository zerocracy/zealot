---
name: review-pull-request
description: |
  Use this skill when the user wants to review a single pull
  request on GitHub with a verdict and inline comments.
---

## Target

Target pull request user named as `<owner>/<repo>#<number>`.
Clone target repository into local working directory before any `gh pr` call.
Pull inside existing clone before any `gh pr` call.
Refuse to review pull request already merged or closed.
Refuse to review pull request whose `mergeable` field reads `CONFLICTING`.
Refuse to review pull request that already carries submitted review.

## Gather

Check out pull request branch locally.
Fetch metadata, including title, body, base branch, and head branch.
Fetch author, linked issues, reviewers, and labels.
Read full diff once to build mental map of change.
Re-read each changed file in full from working tree.
Collect existing review comments so new review does not duplicate feedback.
Collect existing review-summary bodies so new review avoids duplicate feedback.
Check current CI status before judging code.

## Questions

Judge each hunk against four questions in order.
Ask whether hunk matches stated intent in title and body.
Ask whether it respects conventions written in `README.md` and `CLAUDE.md`.
Ask whether it preserves existing invariants of surrounding code.
Ask whether it covers its own edge cases with tests.
Raise inline comment for every `no` answer.

## Defects

Look for common defects first.
Watch for off-by-one, wrong branch, misnamed variable, or unguarded null.
Watch for leaked resource or missing test for new code path.
Watch for contradicted doc or public method without interface declaration.
Watch for introspection or cast where polymorphism would do.
Look for scope creep.
Watch for refactor smuggled into bug fix or rename of unrelated identifiers.
Watch for reformatting of untouched files or unjustified new dependency.

## Comments

Leave one inline comment per defect, anchored to exact file and line.
Never bundle several defects into single comment.
Write each inline comment as few short sentences.
Name defect, explain why it is wrong, and suggest smallest fix.
Use GitHub suggested-change blocks only for one-line patches.
Run full build locally on checked-out branch when project rules expect it.

## Verdict

Decide one verdict from APPROVE, COMMENT, or REQUEST_CHANGES.
Never invent fourth verdict.
Choose APPROVE when no inline comment names real defect.
Choose APPROVE when diff respects conventions and tests cover new behavior.
Choose APPROVE when CI is green.
Choose REQUEST_CHANGES when one comment names correctness defect to fix.
Choose REQUEST_CHANGES when one comment names security or contract defect.
Choose REQUEST_CHANGES when diff causes failure that turns CI red.
Choose COMMENT when inline comments name only stylistic concerns.
Choose COMMENT when inline comments name only scope concerns.

## Summary

Compose review-summary body as one short paragraph.
State verdict, name strongest reason, and point to inline comments by count.
Never duplicate inline comments in summary.
Submit review in one call so comments and verdict land together.

## Limits

Do not merge, auto-merge, close, or push commits to branch.
Do not assign reviewers, change labels, or set milestones.
Do not open new issues or new pull requests as part of this skill.
Name larger problem in review summary and ask author to file follow-up issue.
Do not file follow-up issue on author's behalf.
Stop after you submit review.
Report short factual summary of pull request URL and verdict.
Report inline comment count and CI status at time of review.
