---
name: triage-issue
description: |
  Use this skill to triage a single GitHub issue: read
  the report, decide whether it is a real bug, an
  enhancement request, a question, a duplicate, or
  invalid, pick labels from the labels the repository
  already defines, and apply them with `gh`; close the
  issue when the outcome is `duplicate` or `invalid`,
  otherwise leave it open. Do not assign it, do not
  write a fix — only classify and, when warranted,
  close. One issue per run — then stop.
---

Operate on the GitHub issue named in the user's prompt
  as `<owner>/<repo>#<number>`; refuse to run when no
  target is named.

Refuse to classify an issue that is already closed
  unless the user explicitly asked for a retro-label;
  a closed issue is out of the working backlog, and a
  late label rarely changes the outcome.

Fetch the issue body, the issue title, the existing
  labels, the author login, the assignees, the linked
  pull requests, and every comment on the thread with
  `gh issue view <number> --json` and
  `gh api repos/<owner>/<repo>/issues/<number>/comments`,
  so the classification is grounded in the full thread
  and not in the title alone.

Stop the run immediately if the issue thread already
  carries a comment posted by the same GitHub account
  that runs this skill — resolve the account login with
  `gh api user --jq .login` and compare it against the
  `user.login` field of every comment; a prior comment
  from this account means the issue has already been
  handled on an earlier run, and a second pass would
  duplicate the work.

Fetch the catalogue of labels the repository already
  defines with `gh label list --repo <owner>/<repo> --limit 200`,
  and treat that list as the only vocabulary available
  to this skill; do not invent new labels, do not
  rename existing ones, and do not create a label that
  is missing — ask the user to create it instead.

Map the issue to exactly one primary kind from the
  vocabulary — `bug`, `enhancement`, `question`,
  `duplicate`, `invalid`, or the closest synonym the
  repository defines (`feature`, `improvement`,
  `support`, `wontfix`) — and never apply two primary
  kinds at once, because a single issue has a single
  nature.

Skip the primary-kind label entirely when the report
  does not clearly fit one kind in the vocabulary, when
  the evidence is too thin to support a confident
  choice, or when a firm classification needs the
  project architect's call; an unlabelled issue is a
  valid outcome of this skill, and an honest skip is
  cheaper than a label the maintainer must later
  reverse.

Post a short comment asking the project architect for
  a call when the skip is triggered by the need for a
  firm classification from above — name the question
  the architect needs to answer (`bug or enhancement?`,
  `in scope?`, `is the platform supported?`) in one
  sentence, and `@`-mention the architect by the login
  the repository names in `CODEOWNERS`, the README, or
  a `MAINTAINERS` file — so the issue is not stalled
  while waiting for a label.

Choose `bug` when the report names a behavior that
  contradicts the documented contract, the README, the
  `CLAUDE.md` rules, or a test that the code should
  satisfy; a bug is a divergence from the stated
  intent, not a wish for a new feature.

Choose `enhancement` (or the repository's synonym) when
  the report asks for a new behavior, a new option, a
  new integration, or an improvement to an existing
  feature that the contract does not yet promise; an
  enhancement is a request for scope growth, not a
  defect.

Choose `question` (or the repository's synonym) when
  the report asks how to use the project, how to
  configure it, or how something works, and the answer
  is documentation rather than code; redirect the
  reporter to the README or the discussions forum in
  the same run if the project uses one.

Choose `duplicate` when the report restates an
  already-open or recently-closed issue by symptom,
  file, or proposed fix; list the duplicates the
  repository already carries
  (`gh issue list --search <keywords>`) before applying
  the label, and never apply `duplicate` without
  naming the original issue number in the same run.

Choose `invalid` when the report describes behavior
  the source code does not support, runs on a platform
  the project does not target, or is based on a
  misreading of the documentation; an invalid label
  is a firm answer, not a guess, and it requires the
  evidence to back it.

Close the issue with
  `gh issue close <number> --reason "not planned"` and
  apply no primary label when the report is obvious
  spam — promotional content, an off-topic
  advertisement, a malformed paste with no actionable
  body, or machine-generated noise — and post one short
  comment that names the symptom (the promo link, the
  off-topic subject, the missing body) before the close
  call, without an invitation to re-open the thread.

Add scope labels (`docs`, `tests`, `build`, `ci`,
  `performance`, `security`, `ui`, `api`, ...) on top
  of the primary kind only when the repository defines
  those scopes and the issue clearly fits one of
  them; do not invent a scope the labels do not
  already carry.

Add a severity label (`critical`, `high`, `medium`,
  `low`, or the repository's equivalent) only when the
  repository uses a severity vocabulary and the
  evidence in the issue justifies the level; an
  unguarded severity guess inflates the backlog.

Add a `good first issue` label only when the fix is
  small, the file is named in the report, the
  reproduction is clear, and the repository already
  uses the label; never apply it to a vague or
  speculative report.

Add a `needs-reproduction` label (or the repository's
  synonym) when the report names a symptom but does not
  name a version, a platform, a command, or a stack
  trace that lets a maintainer reproduce it; the
  label asks the reporter for the missing context.

Never apply labels that demand action by a specific
  person (`assigned:<login>`, `claimed`, `in-progress`,
  `blocked-on-<login>`); those are workflow signals
  the maintainer owns, not classification signals this
  skill controls.

Never apply priority labels (`P0`, `P1`, `P2`,
  `priority:high`) unless the user explicitly asked
  for a priority on the run; priority is a planning
  call the maintainer makes from a full backlog, not a
  triage call.

Post a comment on the issue before applying the labels
  only when the comment adds value the thread does not
  already carry — a concrete piece of criticism, a
  concrete piece of support, a missing piece of
  evidence, or a risk a future implementor would
  otherwise miss — and keep the tone neutral, because
  the job of the comment is to help the next reader
  decide how to resolve the issue, not to praise the
  reporter or to dismiss the report.

Stay silent when there is nothing important to add:
  silence is a valid outcome of this skill, and a run
  that applies labels without a comment is a complete
  run when the thread already carries the relevant
  evidence and a fresh comment would only restate it.

React with a GitHub emoji to a comment in the thread —
  or to the issue body itself — when the agent would
  otherwise want to praise or push back on it: pick
  `+1` for a position the agent agrees with, `-1` for
  one the agent disagrees with, `heart` for a
  contribution that supplies evidence the maintainer
  should not miss, `hooray` for a correct resolution
  the reporter has already proposed, and `confused`
  for a statement that contradicts the thread or the
  source code; post the reaction with
  `gh api -X POST /repos/<owner>/<repo>/issues/comments/<id>/reactions -f content=<reaction>`
  for a comment, or
  `gh api -X POST /repos/<owner>/<repo>/issues/<number>/reactions -f content=<reaction>`
  for the issue body, because a single emoji carries
  the signal a redundant prose reply would carry
  without cluttering the thread.

React on every comment the agent has a stance on, not
  only the ones that trigger a prose comment; a
  reaction is the agent's default channel for like and
  dislike, and skipping it leaves the maintainer blind
  to the agent's read of the thread.

Skip the reaction on text that carries neither
  agreement nor disagreement — a neutral status update,
  a bot post, a quote of the title, a routine
  cross-link — because a reaction without a reason
  adds noise.

Never post a comment that summarizes the issue back to
  the reporter, thanks the reporter, restates the
  chosen label, or acknowledges receipt; such a comment
  adds no value and clutters the thread, and a `+1`
  reaction covers the same intent at lower cost.

Ground the comment, when one is posted, in evidence
  from the thread and the source code: name the file,
  the function, the test, or the change a maintainer
  would touch; cite the README, the contract, or the
  code when the issue is a question; and call out the
  risks — broken tests, behavior changes, backward
  incompatibility, security exposure, performance
  regressions — that the resolution would carry.

Apply the chosen labels in one call with
  `gh issue edit <number> --add-label "<label1>,<label2>,..."`
  so the change lands as a single edit, and remove
  contradictory labels in the same call with
  `--remove-label` rather than across two writes.

Close the issue with
  `gh issue close <number> --repo <owner>/<repo>
  --reason "not planned"` when the chosen primary
  kind is `duplicate` or `invalid`, because such an
  issue carries no actionable work and a closed state
  keeps the backlog honest; leave the issue open for
  every other primary kind.

Post a closing comment before the close call that
  states the reason in one or two sentences — for
  `duplicate`, name the original issue number and
  explain the overlap; for `invalid`, cite the file,
  the documentation, or the platform constraint that
  rules the report out — and end the comment with a
  line inviting the reporter to open a new issue if
  the problem persists or a similar problem appears,
  so the close lands as an explanation rather than a
  dismissal.

Do not reopen an issue, do not merge it into another
  issue, do not edit its title or body, do not assign
  it to anyone, do not set a milestone, and do not add
  it to a project board; the only write actions
  permitted by this skill are the label edit, the
  classification comment, the optional follow-up
  comment, and — for `duplicate` or `invalid` — the
  close.

Post one short follow-up comment on the issue only
  when the classification calls for action by the
  reporter — for `duplicate`, name the original issue
  and ask the reporter to subscribe to it; for
  `needs-reproduction`, name the missing context and
  ask the reporter to add it; for `question`, point
  the reporter at the doc or the forum.

Keep the follow-up comment to one or two sentences of
  plain prose, because the comment is a pointer, not a
  second report.

Stop after the labels are applied, the optional
  follow-up comment is posted, and — when the outcome
  was `duplicate` or `invalid` — the issue is closed:
  do not classify a second issue, do not start a fix,
  and re-run this skill from the top for the next
  issue.

Report at the end of the run a short factual summary:
  the issue URL, the primary kind chosen, the scope
  and severity labels applied, the labels removed (if
  any), the close state (closed or left open) with the
  reason when closed, and the link to the follow-up
  comment when one was posted, so the maintainer can
  scan the result without re-reading the whole thread.
