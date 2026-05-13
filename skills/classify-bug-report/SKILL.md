---
name: classify-bug-report
description: |
  Use this skill to triage a single GitHub issue: read
  the report, decide whether it is a real bug, an
  enhancement request, a question, a duplicate, or
  invalid, pick labels from the labels the repository
  already defines, and apply them with `gh`. Do not
  close the issue, do not assign it, do not write a
  fix — only classify. One issue per run — then stop.
argument-hint: "[owner/repo] [issue-number]"
user-invokable: false
---

Operate on the GitHub issue `$1#$2`.

Refuse to classify a pull request as if it were an
  issue; this skill labels bug reports, not change
  proposals, and the `review-pull-request` skill exists
  for the other case.

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

Stop the run immediately if the issue already carries a
  primary kind label (`bug`, `enhancement`, `question`,
  `duplicate`, `invalid`, or the repository's synonym
  such as `feature`, `improvement`, `support`,
  `wontfix`); a classified issue is out of this skill's
  scope, and overriding a prior triage decision belongs
  to the maintainer.

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

Post one comment on the issue before applying the
  labels: explain in plain prose how the bug or the
  enhancement could be fixed — name the file, the
  function, or the change a maintainer would touch when
  the evidence in the thread allows it — and answer the
  question directly when the issue is a question,
  citing the README or the source code rather than
  guessing.

Apply the chosen labels in one call with
  `gh issue edit <number> --add-label "<label1>,<label2>,..."`
  so the change lands as a single edit, and remove
  contradictory labels in the same call with
  `--remove-label` rather than across two writes.

Do not close the issue, do not reopen it, do not
  merge it into another issue, do not edit its title
  or body, do not assign it to anyone, do not set a
  milestone, and do not add it to a project board; the
  only write action permitted by this skill is the
  label edit.

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

Stop after the labels are applied and the optional
  follow-up comment is posted: do not classify a
  second issue, do not start a fix, and re-run this
  skill from the top for the next issue.

Report at the end of the run a short factual summary:
  the issue URL, the primary kind chosen, the scope
  and severity labels applied, the labels removed (if
  any), and the link to the follow-up comment when
  one was posted, so the maintainer can scan the
  result without re-reading the whole thread.
