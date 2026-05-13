---
name: make-one-contribution
description: Use this skill as the daily entry point for a single contribution across a list of GitHub repositories and accounts — first scans the notification inbox for an inbound comment from another user and delegates to respond-to-comment, otherwise picks one repository from the list and one of file-bug-report, classify-bug-report, submit-pull-request, or review-pull-request based on backlog and queue signals, then delegates and stops.
---

Read `CONVENTIONS.md` at the root of this plugin
  before doing anything else, hold every rule in that
  file as binding for the rest of the run, and pass it
  to the delegated sub-skill as the canonical source
  of voice, signature, tooling, and Markdown rules.

Read the input as two lists named in the user's
  prompt: a list of repositories in `<owner>/<repo>`
  form, and a list of GitHub accounts in `<login>`
  form; refuse to run when neither list is provided,
  and do not invent entries from memory.

Treat the account list as the set of GitHub logins
  whose notifications, mentions, and review requests
  this run is allowed to act on; the current login
  must appear in the list, and the run skips any
  notification addressed only to a login outside the
  list.

Treat the repository list as the set of repositories
  this run is allowed to touch as a target; ignore
  any notification, issue, or pull request whose
  repository is not on the list, even when the
  current login is mentioned, because the run is not
  authorized to write outside the list.

Identify the current GitHub login once with
  `gh api user --jq .login` and capture it for the
  rest of the run, because every check that filters
  comments, pull requests, or notifications by author
  or recipient depends on this value.

## Step 1: scan the notification inbox first

Fetch the unread notifications addressed to the
  current login with
  `gh api notifications --paginate
  -f participating=true -f all=false`,
  because an inbound question from another user
  outranks every other contribution type on this run.

Filter the notification list down to entries whose
  `repository.full_name` is on the repository list,
  whose `reason` is one of `mention`,
  `review_requested`, `comment`, or `author`, and
  whose `subject.type` is `Issue`, `PullRequest`, or
  `PullRequestReview`, because those are the four
  shapes of an inbound comment this skill is allowed
  to answer.

For each remaining notification, fetch the source
  comment from the matching endpoint — inline review
  comments
  (`gh api repos/<owner>/<repo>/pulls/<number>/comments`),
  review-summary bodies
  (`gh api repos/<owner>/<repo>/pulls/<number>/reviews`),
  or issue-level comments
  (`gh api repos/<owner>/<repo>/issues/<number>/comments`)
  — and locate the most recent comment authored by a
  login other than the current login that mentions
  the current login or sits on a thread the current
  login already participated in.

Discard a notification when the latest comment on the
  thread was authored by the current login, by the
  bot that opened the pull request, by
  `github-actions[bot]` when the body is a pure CI
  status echo, or by an account on neither the
  account list nor the recognized automated-reviewer
  set (`copilot-pull-request-reviewer`,
  `coderabbitai`, `sonarcloud`, `codecov`).

Discard a notification when the latest comment from
  another user has already been answered in-thread by
  the current login with either a reply or a
  referenced commit; an emoji reaction does not count
  as an answer.

Choose `respond-to-comment` as the contribution type
  when at least one notification survives the
  filters; pick the oldest surviving comment as the
  target and pass the artifact reference
  `<owner>/<repo>#<number>` together with the comment
  id to the sub-skill, then stop the decision loop
  and delegate.

## Step 2: pick a repository when the inbox is clean

Walk the repository list in the order it was given
  and, for each repository, fetch the same four
  counts the decision below depends on, because the
  repository chosen is the one with the strongest
  signal across these counts.

Fetch the open issues with
  `gh issue list --repo <owner>/<repo> --state open
  --limit 200 --json
  number,title,labels,assignees,createdAt`,
  and count them; the total — the backlog size —
  feeds the `file-bug-report` and `classify-bug-report`
  branches.

Count the unlabeled open issues separately — those
  whose `labels` array is empty — because they are
  the trigger for the `classify-bug-report` branch.

Fetch the open pull requests with
  `gh pr list --repo <owner>/<repo> --state open
  --limit 200 --json number,title,author,createdAt`,
  and count them; the total — the pull request queue
  size — feeds the `submit-pull-request` and
  `review-pull-request` branches.

Pick a single repository from the list before picking
  a contribution type: prefer the repository with at
  least one unlabeled open issue, then the
  repository with ten or fewer open issues, then the
  repository with five or fewer open pull requests,
  and otherwise the repository with the largest open
  pull request queue, because each branch of the
  decision below already maps to one of those
  signals.

## Step 3: pick the contribution type for the chosen repo

Choose `classify-bug-report` when at least one open
  issue in the chosen repository carries no labels at
  all, because an unlabeled issue is invisible to
  triage and a single label is the cheapest
  contribution available on this pass.

Choose `file-bug-report` when the backlog of the
  chosen repository is short — ten or fewer open
  issues — because a short backlog means the
  repository can absorb a fresh bug report without
  drowning the maintainer in untriaged noise.

Choose `submit-pull-request` when the open pull
  request queue of the chosen repository is short —
  five or fewer open pull requests — because a short
  queue means a new fix will be reviewed and merged
  rather than buried under older pull requests.

Choose `review-pull-request` otherwise, because a
  long pull request queue is the strongest signal
  that the repository needs reviewers more than it
  needs new reports, fixes, or labels.

Evaluate the conditions in the order listed above
  and pick the first one that matches; do not score
  the conditions, do not combine them, and do not
  weight them — the order is the decision, and
  `respond-to-comment` from Step 1 always wins when
  it matches.

## Step 4: delegate

Pick a single target inside the chosen contribution
  type before delegating: for `respond-to-comment`,
  pass the artifact reference and the comment id
  selected in Step 1; for `file-bug-report`, no
  target is needed because the sub-skill picks the
  defect itself; for `classify-bug-report`, pick the
  oldest unlabeled open issue in the chosen
  repository; for `submit-pull-request`, pick the
  oldest open issue labeled `bug` or `enhancement`
  in the chosen repository that is unassigned and
  has no claimed work in the thread; for
  `review-pull-request`, pick the oldest open pull
  request in the chosen repository not authored by
  the current login.

Delegate to the chosen sub-skill with the picked
  target as input, and let that sub-skill enforce
  its own contract — duplicate checks, build steps,
  comment style, stop conditions — without
  re-implementing those rules here.

Do not invoke more than one sub-skill in the same
  run, do not chain a second contribution after the
  first completes, and do not retry the same
  sub-skill after it returns — re-run this skill
  from the top on the next pass.

Do not fall back to a different contribution type
  when the chosen sub-skill reports no actionable
  target — for example, when `file-bug-report`
  finds no defect or when `review-pull-request`
  finds no reviewable pull request; the contract of
  this skill is one decision per run, and the
  fallback belongs to the next run.

Do not fall back to a different repository when the
  sub-skill reports no actionable target inside the
  chosen repository; the chosen repository was the
  decision, and the next run picks again from the
  whole list.

Stop after the delegated sub-skill finishes: do not
  pick a second repository, do not pick a second
  contribution type, do not start a fix, a review, a
  classification, or a report on top of the first,
  and re-run this skill from the top for the next
  pass.

Report at the end of the run a short factual
  summary: the repository chosen, the contribution
  type chosen, the reason this branch of the
  decision was taken (the matched notification, the
  unlabeled issue count, the backlog size, or the
  pull request queue size), the target picked, and
  the URL of the artifact created or edited by the
  delegated sub-skill — the reply posted on the
  inbound comment, the new issue, the labeled
  issue, the new pull request, or the review.
