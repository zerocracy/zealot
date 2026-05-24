---
name: review-pull-request
description: |
  Use this skill to review a single pull request on GitHub
  with a verdict and inline comments.
---

Target the pull request the user named as `<owner>/<repo>#<number>`.
Clone the target repository into a local working directory or pull inside an existing clone before any `gh pr` call.
Refuse to review a pull request that is already merged or closed.
Refuse to review a pull request whose `mergeable` field reads `CONFLICTING`.
Refuse to review a pull request that already carries a submitted review from another reviewer.
Check out the pull request branch locally.
Fetch the metadata, including title, body, base branch, head branch, author, linked issues, reviewers, and labels.
Read the full diff once to build a mental map of the change.
Re-read each changed file in full from the working tree.
Collect existing review comments and review-summary bodies so the new review does not duplicate feedback.
Check the current CI status before judging the code.
Judge each hunk against four questions in order.
Ask whether the hunk matches the stated intent in the title and body.
Ask whether it respects the conventions written in `README.md` and `CLAUDE.md`.
Ask whether it preserves the existing invariants of the surrounding code.
Ask whether it covers its own edge cases with tests.
Raise an inline comment for every `no` answer.
Look for common defects first, like an off-by-one, a wrong branch, a misnamed variable, an unguarded null, a leaked resource, a missing test for a new code path, a contradicted doc, a public method without an interface declaration, or an introspection or cast where polymorphism would do.
Look for scope creep, like a refactor smuggled into a bug fix, a rename of unrelated identifiers, reformatting of files not touched by the stated change, or an unjustified new dependency.
Leave one inline comment per defect, anchored to the exact file and line.
Never bundle several defects into a single comment.
Write each inline comment as a few short sentences naming the defect, explaining why it is wrong, and suggesting the smallest change that resolves it.
Use GitHub suggested-change blocks only for one-line patches the author can apply with a single click.
Run the full build locally on the checked-out branch when the project's rules expect it.
Decide one verdict from APPROVE, COMMENT, or REQUEST_CHANGES, and never invent a fourth.
Choose APPROVE when no inline comment names a real defect, the diff respects project conventions, the tests cover the new behavior, and CI is green.
Choose REQUEST_CHANGES when at least one inline comment names a correctness, security, or contract defect that must be fixed before merge, or when CI is red on a failure the diff causes.
Choose COMMENT when the inline comments name only stylistic or scope concerns.
Compose the review-summary body as one short paragraph stating the verdict, naming the strongest reason, and pointing to the inline comments by count.
Never duplicate the inline comments in the summary.
Submit the review in one call so the inline comments and the verdict land together.
Do not merge, auto-merge, close, push commits to the branch, assign reviewers, change labels, or set milestones.
Do not open new issues or new pull requests as part of this skill.
Name a larger problem in the review summary and ask the author to file a follow-up issue, but do not file it on their behalf.
Stop after the review is submitted.
Report a short factual summary of the pull request URL, the verdict, the inline comment count, and the CI status at the time of review.
