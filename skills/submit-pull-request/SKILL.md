---
name: submit-pull-request
description: |
  Use this skill to open a single pull request against
  a GitHub repository.
---

Target the GitHub repository the user named.
Operate from the feature branch the master skill prepared in the local working tree.
Identify the base branch from the repository's actual default, not a hard-coded `main` or `master`.
Fetch and rebase the branch onto the latest base before opening the pull request.
Resolve every rebase conflict by hand.
Re-run the build after each conflict resolution.
Never use `git rebase --skip` or `-X ours` and `-X theirs` shortcuts.
Run the full build, every test, every linter, and every static check named in `README.md` or `CLAUDE.md` before pushing the branch.
Stabilize the build before pushing the branch.
Push the branch and confirm the push succeeded before opening the pull request.
Compose the title as a short, declarative sentence in the imperative mood, lowercase except for proper nouns, under 72 characters, with no trailing punctuation.
Prepend the ticket number with a `#` to the title when the pull request closes or refers to an issue.
Match the title to the dominant commit type in the branch when the project's commit style expects it.
Write the body as three short paragraphs covering the motivation and the linked issue, the visible effect of the change, and the checks that were run with their outcome.
Add a `Closes #<number>` or `Fixes #<number>` line on its own when the pull request resolves an open issue.
Ground the test-plan paragraph in commands that actually ran on the local machine.
Keep the body compact.
Set the base and head branches explicitly even when they match the defaults.
Mark the pull request as draft only when the user explicitly requested a draft.
Never mark it ready-for-review automatically.
Do not request reviewers, assign labels, set milestones, or assign the pull request unless the user explicitly asked.
Do not merge, approve, auto-merge, or force-push the pull request after it is open.
Capture the pull request URL and watch CI once so a first-build failure surfaces inside the run.
Fix CI failures inside this run by committing and pushing additional commits to the same branch.
Do not consider the pull request submitted until every required check is green or the user accepts a red status.
Stop as soon as every required CI check on the latest commit is green.
Do not poll for reviewer comments, reviews, or approvals after CI turns green.
Report a short factual summary of the pull request URL, the base and head branches, the commits pushed during the run, and the final CI status.
