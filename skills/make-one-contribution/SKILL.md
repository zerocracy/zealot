---
name: make-one-contribution
description: Use this skill as the daily entry point for a single contribution to a GitHub repository — picks one of file-bug-report, classify-bug-report, submit-pull-request, review-pull-request, or address-reviewer-comments based on backlog and queue signals, then delegates and stops.
---

Read `CONVENTIONS.md` at the root of this plugin before
  doing anything else, hold every rule in that file as
  binding for the rest of the run, and pass it to the
  delegated sub-skill as the canonical source of voice,
  signature, tooling, and Markdown rules.

Operate on the GitHub repository named in the user's
  prompt as the target; refuse to run when no target is
  named, and do not invent one from memory.

Read the `README.md` and `CLAUDE.md` files at the root
  of the repository before deciding the contribution
  type; they name the project's scope, its supported
  platforms, and the conventions every sub-skill in
  this run must respect.

Fetch the open issues with
  `gh issue list --repo <owner>/<repo> --state open --limit 200 --json number,title,labels,assignees,createdAt`,
  and count them; the total — the backlog size — drives
  the first branch of the decision.

Count the unlabeled open issues separately — those
  whose `labels` array is empty — because they are the
  trigger for the second branch of the decision.

Fetch the open pull requests with
  `gh pr list --repo <owner>/<repo> --state open --limit 200 --json number,title,author,createdAt`,
  and count them; the total — the pull request queue
  size — drives the third branch of the decision.

Identify the current GitHub login once with
  `gh api user --jq .login` and capture it for the rest
  of the run, because every check that filters pull
  requests by author depends on this value.

Fetch the open pull requests authored by the current
  login with
  `gh pr list --repo <owner>/<repo> --state open --author @me --limit 100 --json number,title,createdAt,updatedAt`,
  because a pull request submitted on an earlier run
  may still carry unanswered reviewer comments, and
  finishing that thread ranks above every other
  contribution type on this pass.

Walk the list of own open pull requests oldest first
  and, for each one, collect reviewer feedback from the
  same three sources `address-reviewer-comments` uses
  — inline review comments
  (`gh api repos/<owner>/<repo>/pulls/<number>/comments`),
  review-summary bodies
  (`gh api repos/<owner>/<repo>/pulls/<number>/reviews`),
  and issue-level comments
  (`gh api repos/<owner>/<repo>/issues/<number>/comments`)
  — so the count of unanswered feedback agrees with
  what the sub-skill will see when it runs.

Treat as not-reviewer-feedback any comment authored by
  the pull request author themselves, by the bot that
  opened the pull request, or by `github-actions[bot]`
  when the body is a pure CI status echo; every other
  comment counts as reviewer feedback.

Treat a comment as unanswered when no later in-thread
  reply from the pull request author exists on the same
  thread; an emoji reaction does not count as an
  answer, and a resolved-but-unreplied thread does not
  count as an answer.

Choose `address-reviewer-comments` first when at least
  one open pull request authored by the current login
  carries at least one unanswered reviewer comment,
  because finishing an in-flight review ranks above
  starting a new contribution on top of a stalled one.

Choose `file-bug-report` when the backlog is short —
  ten or fewer open issues — because a short backlog
  means the repository can absorb a fresh bug report
  without drowning the maintainer in untriaged noise.

Choose `classify-bug-report` when at least one open
  issue carries no labels at all, because an unlabeled
  issue is invisible to triage and a single label is
  the cheapest contribution available on this pass.

Choose `submit-pull-request` when the open pull request
  queue is short — five or fewer open pull requests —
  because a short queue means a new fix will be reviewed
  and merged rather than buried under older pull requests.

Choose `review-pull-request` otherwise, because a long
  pull request queue is the strongest signal that the
  repository needs reviewers more than it needs new
  reports, fixes, or labels.

Evaluate the five conditions in the order listed above
  and pick the first one that matches; do not score the
  conditions, do not combine them, and do not weight
  them — the order is the decision, and
  `address-reviewer-comments` always wins when it
  matches.

Pick a single target inside the chosen contribution
  type before delegating: for `address-reviewer-comments`,
  pick the oldest open pull request authored by the
  current login that carries at least one unanswered
  reviewer comment; for `file-bug-report`, no target is
  needed because the sub-skill picks the defect itself;
  for `classify-bug-report`, pick the oldest unlabeled
  open issue; for `submit-pull-request`, pick the oldest
  open issue labeled `bug` or `enhancement` that is
  unassigned and has no claimed work in the thread; for
  `review-pull-request`, pick the oldest open pull
  request not authored by the account running this
  skill.

Delegate to the chosen sub-skill with the picked target
  as input, and let that sub-skill enforce its own
  contract — duplicate checks, build steps, comment
  style, stop conditions — without re-implementing
  those rules here.

Do not invoke more than one sub-skill in the same run,
  do not chain a second contribution after the first
  completes, and do not retry the same sub-skill after
  it returns — re-run this skill from the top on the
  next pass.

Do not fall back to a different contribution type when
  the chosen sub-skill reports no actionable target —
  for example, when `file-bug-report` finds no defect
  or when `review-pull-request` finds no reviewable
  pull request; the contract of this skill is one
  decision per run, and the fallback belongs to the
  next run.

Stop after the delegated sub-skill finishes: do not
  pick a second repository, do not pick a second
  contribution type, do not start a fix, a review, a
  classification, or a report on top of the first, and
  re-run this skill from the top for the next pass.

Report at the end of the run a short factual summary:
  the repository, the contribution type chosen, the
  reason this branch of the decision was taken (the
  count of own open pull requests with unanswered
  reviewer comments, the backlog size, the unlabeled
  issue count, or the pull request queue size), the
  target picked, and the URL of the artifact created
  or edited by the delegated sub-skill — the pull
  request whose comments were addressed, the new
  issue, the labeled issue, the new pull request, or
  the review.
