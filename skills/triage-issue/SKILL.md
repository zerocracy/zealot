---
name: triage-issue
description: |
  Use this skill when the user wants to triage a single
  GitHub issue by classifying it and applying labels from
  the repository's existing label vocabulary.
---

## Scope

Target GitHub issue user named as `<owner>/<repo>#<number>`.
Refuse to classify issue already closed unless user asked for retro-label.
Fetch issue body, title, labels, author, assignees, and linked pull requests.
Fetch every comment on issue.

## Skip

Stop when thread already carries comment from same account running this skill.
Stop when issue author is same account running this skill.

## Vocabulary

Fetch repository's existing label vocabulary.
Treat it as only vocabulary allowed.
Do not invent new labels, rename existing ones, or create missing ones.
Ask user to create missing label instead.

## Kind

Map issue to exactly one primary kind.
Choose from `bug`, `enhancement`, `question`, `duplicate`, or `invalid`.
Or choose repository's closest synonym.
Synonyms include `feature`, `improvement`, `support`, or `wontfix`.
Never apply two primary kinds at once.

## Skip Kind

Skip primary-kind label when report does not clearly fit one kind.
Skip primary-kind label when evidence is too thin.
Skip primary-kind label when call belongs to project architect.
Post short comment asking project architect for call when skip needs input.
Mention architect by login repository names in `CODEOWNERS`.
Or mention architect by login named in README or `MAINTAINERS` file.

## Bug

Choose `bug` when report names behavior that contradicts documented contract.
Or choose `bug` when report contradicts README or `CLAUDE.md` rules.
Or choose `bug` when report contradicts test that code must satisfy.

## Enhancement

Choose `enhancement` when report asks for new behavior, option, or integration.
Or choose `enhancement` when report asks for improvement to existing feature.

## Question

Choose `question` when report asks how to use project.
Or choose it when report asks how to configure or understand project.
Redirect reporter to README or discussions forum for question.

## Duplicate

Choose `duplicate` when report restates open or recently-closed issue.
Match by symptom, file, or proposed fix.
Name original issue number when applying `duplicate`.

## Invalid

Choose `invalid` when report describes behavior source code does not support.
Or choose `invalid` when report runs on platform project does not target.
Or choose `invalid` when report rests on misreading of documentation.

## Spam

Close issue with reason `not planned` and apply no primary label for spam.
Post one short comment naming spam symptom before close call.

## Scope Labels

Add scope labels only when repository defines them and issue clearly fits.
Scope labels include `docs`, `tests`, `build`, `ci`, `performance`.
Scope labels also include `security`, `ui`, or `api`.
Add severity label only when repository uses severity vocabulary.
Add severity label only when evidence justifies level.

## Other Labels

Add `good first issue` label only when fix is small.
Add it only when report names file and gives clear reproduction.
Add it only when repository already uses label.
Add `needs-reproduction` label when report names symptom but no version.
Or add it when report names no platform, command, or stack trace.
Never apply labels that demand action by specific person.
Never apply priority labels unless user explicitly asked.

## Comments

Post comment only when comment adds value thread does not already carry.
Keep comment tone neutral.
Stay silent when there is nothing important to add.
Never post comment that summarizes issue back or thanks reporter.
Never post comment that restates chosen label or acknowledges receipt.

## Reactions

React with `+1`, `-1`, `heart`, `hooray`, or `confused` on stance.
React when agent would otherwise praise or push back on comment or issue body.
React on every comment agent has stance on.
React beyond only ones that trigger prose comment.
Skip reaction on neutral status updates or bot posts.
Skip reaction on quoted titles or routine cross-links.

## Evidence

Ground every prose comment in evidence from thread and source code.
Name file, function, test, or change maintainer would touch.
Cite README, contract, or code when issue is question.
Call out risks resolution would carry.
Risks include broken tests, behavior changes, or backward incompatibility.
Risks also include security exposure or performance regressions.

## Edits

Pass `--repo <owner>/<repo>` to every `gh issue view` call.
Pass `--repo <owner>/<repo>` to every `gh issue edit` call.
Pass `--repo <owner>/<repo>` to every `gh issue close` call.
Apply chosen labels in one call so change lands as single edit.
Remove contradictory labels in same call rather than across two writes.

## Closing

Close issue with reason `not planned` when primary kind is `duplicate`.
Or close it when primary kind is `invalid`.
Leave issue open for every other primary kind.
Post closing comment before close call.
State reason in one or two sentences.
Name original issue number when closing as `duplicate`.
Cite file, documentation, or platform constraint when closing as `invalid`.
End closing comment with invitation to open new issue if problem persists.

## Restraint

Do not reopen issue, merge it into another, or edit its title or body.
Do not assign it, set milestone, or add it to project board.

## Follow-up

Post one short follow-up comment only when classification calls for action.
Action means reporter must act.
Keep follow-up comment to one or two sentences.

## Finish

Stop after you apply labels and post optional follow-up comment.
Stop after issue is closed when warranted.
Report short factual summary of issue URL and primary kind chosen.
Report scope and severity labels applied, and labels removed.
Report close state and reason, and link to follow-up comment.
