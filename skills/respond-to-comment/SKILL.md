---
name: respond-to-comment
description: |
  Use this skill to answer a single GitHub comment
  addressed to the current login, with either a fix
  or a push-back reply on the same thread.
---

Target the comment the user named as `<owner>/<repo>#<number>` together with a comment identifier.
Stop when the working directory has any uncommitted changes, untracked files, or staged hunks.
Identify the current GitHub login once and capture it for the run.
Fetch the target pull request or issue metadata before touching anything.
Check out the pull request branch locally for a pull request comment, and pull the latest state.
Stay on the default branch for an issue comment.
Fetch the comment in question and the surrounding thread from the matching endpoint.
Locate the exact comment by its id.
Discard the run when the comment is authored by the pull request author, the issue author, the bot that opened the pull request, or `github-actions[bot]` echoing CI status.
Treat suggestions from automated reviewers like `copilot-pull-request-reviewer`, `coderabbitai`, `sonarcloud`, and `codecov` as comments to answer.
Skip the run when the comment has already been answered in-thread by the current login with a reply or a referenced commit.
An emoji reaction does not count as an answer.
Read the surrounding code at the exact file and line the comment points to for a pull request comment.
Read the issue body, every prior comment, the linked pull requests, and the referenced files before deciding anything for an issue comment.
Classify the comment as either FIX or PUSH-BACK, and never invent a third.
Choose FIX when the comment names a real defect, a violated convention, a missing test, a broken invariant, a typo, a security or correctness issue, or any concrete improvement in scope.
Choose PUSH-BACK when the comment asks for work out of scope, contradicts the conventions in `README.md` or `CLAUDE.md`, restates a personal style preference with no technical basis, misreads the diff or the issue, or duplicates feedback already addressed.
Prefer FIX over PUSH-BACK when the right outcome is genuinely unclear.
For a FIX on a pull request, edit the source by hand and change only what the comment demands.
Do not refactor, rename, or clean up unrelated style under cover of the same edit.
For a FIX on an issue, gather and post the missing information the reporter asked for on the same thread.
Do not silently edit the issue body to insert it.
Run the full build after each source edit and stabilize it before committing.
Every previously-passing test must still pass, every linter must be clean, and no new warning may appear.
Commit each FIX on its own with a message that references the comment.
Do not bundle several unrelated FIXes into one commit.
Push the branch after the commit.
Reply to the comment on the same thread no matter the outcome.
For a FIX on a pull request, post a short reply naming the commit SHA and what changed.
For a FIX on an issue, post the requested information or a link to the follow-up pull request.
For a PUSH-BACK, post a short reply stating the technical reason, citing the project rule, the scope boundary, or the misreading.
Use the GitHub reply endpoints to keep the reply on the original thread.
Start every reply with `@<login>` of the comment author.
Keep every reply short and factual.
Ground every PUSH-BACK reply in a concrete argument like a cited project rule, a scope boundary, a misread diff, or a prior comment.
A bare `I disagree` does not count as an argument.
Do not mark a thread resolved without first posting a reply.
Do not mark a thread resolved on behalf of the reporter when the resolution is in dispute.
Watch CI on a fix commit and do not consider the comment answered until every check is green.
Fix CI failures unrelated to the current comment inside the same run.
Do not approve, request changes, merge, rebase, force-push, close, reopen, relabel, or reassign the artifact.
Do not open new issues, pull requests, or branches as part of this skill.
Stop after the comment is answered with a commit-and-reply pair or a PUSH-BACK reply.
Stop only when CI on the latest commit is green for a pull request fix.
Report a short factual summary of the comment URL, the outcome, the commit SHA, the reply URL, and the final CI status when the artifact was a pull request.
