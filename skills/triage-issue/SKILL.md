---
name: triage-issue
description: |
  Use this skill to triage a single GitHub issue by
  classifying it and applying labels from the
  repository's existing label vocabulary.
---

Target the GitHub issue the user named as `<owner>/<repo>#<number>`.
Refuse to classify an issue that is already closed unless the user asked for a retro-label.
Fetch the issue body, title, labels, author, assignees, linked pull requests, and every comment.
Stop when the thread already carries a comment from the same account running this skill.
Stop when the issue author is the same account running this skill.
Fetch the repository's existing label vocabulary and treat it as the only vocabulary allowed.
Do not invent new labels, rename existing ones, or create missing ones.
Ask the user to create a missing label instead.
Map the issue to exactly one primary kind from `bug`, `enhancement`, `question`, `duplicate`, `invalid`, or the repository's closest synonym like `feature`, `improvement`, `support`, or `wontfix`.
Never apply two primary kinds at once.
Skip the primary-kind label when the report does not clearly fit one kind, the evidence is too thin, or the call belongs to the project architect.
Post a short comment asking the project architect for a call when the skip needs an architect's input.
`@`-mention the architect by the login the repository names in `CODEOWNERS`, the README, or a `MAINTAINERS` file.
Choose `bug` when the report names behavior that contradicts the documented contract, the README, the `CLAUDE.md` rules, or a test the code should satisfy.
Choose `enhancement` when the report asks for a new behavior, option, integration, or improvement to an existing feature.
Choose `question` when the report asks how to use the project, configure it, or understand it.
Redirect the reporter to the README or the discussions forum for a question.
Choose `duplicate` when the report restates an already-open or recently-closed issue by symptom, file, or proposed fix.
Name the original issue number when applying `duplicate`.
Choose `invalid` when the report describes behavior the source code does not support, runs on a platform the project does not target, or is based on a misreading of the documentation.
Close the issue with reason `not planned` and apply no primary label when the report is obvious spam.
Post one short comment naming the spam symptom before the close call.
Add scope labels like `docs`, `tests`, `build`, `ci`, `performance`, `security`, `ui`, or `api` only when the repository defines them and the issue clearly fits.
Add a severity label only when the repository uses a severity vocabulary and the evidence justifies the level.
Add a `good first issue` label only when the fix is small, the file is named in the report, the reproduction is clear, and the repository already uses the label.
Add a `needs-reproduction` label when the report names a symptom but no version, platform, command, or stack trace.
Never apply labels that demand action by a specific person.
Never apply priority labels unless the user explicitly asked.
Post a comment on the issue only when the comment adds value the thread does not already carry.
Keep the comment tone neutral.
Stay silent when there is nothing important to add.
React with `+1`, `-1`, `heart`, `hooray`, or `confused` when the agent would otherwise want to praise or push back on a comment or the issue body.
React on every comment the agent has a stance on, not only the ones that trigger a prose comment.
Skip the reaction on neutral status updates, bot posts, quoted titles, or routine cross-links.
Never post a comment that summarizes the issue back, thanks the reporter, restates the chosen label, or acknowledges receipt.
Ground every prose comment in evidence from the thread and the source code.
Name the file, function, test, or change a maintainer would touch.
Cite the README, the contract, or the code when the issue is a question.
Call out the risks the resolution would carry, like broken tests, behavior changes, backward incompatibility, security exposure, or performance regressions.
Pass `--repo <owner>/<repo>` to every `gh issue view`, `gh issue edit`, and `gh issue close` call.
Apply the chosen labels in one call so the change lands as a single edit.
Remove contradictory labels in the same call rather than across two writes.
Close the issue with reason `not planned` when the primary kind is `duplicate` or `invalid`.
Leave the issue open for every other primary kind.
Post a closing comment before the close call stating the reason in one or two sentences.
Name the original issue number when closing as `duplicate`.
Cite the file, documentation, or platform constraint when closing as `invalid`.
End the closing comment with an invitation to open a new issue if the problem persists.
Do not reopen the issue, merge it into another, edit its title or body, assign it, set a milestone, or add it to a project board.
Post one short follow-up comment only when the classification calls for action by the reporter.
Keep the follow-up comment to one or two sentences.
Stop after the labels are applied, the optional follow-up comment is posted, and the issue is closed when warranted.
Report a short factual summary of the issue URL, the primary kind chosen, the scope and severity labels applied, the labels removed, the close state and reason, and the link to the follow-up comment.
