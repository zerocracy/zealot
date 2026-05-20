---
name: make-one-contribution
description: Use this skill as the daily entry point for a contribution chain against a list of GitHub orgs and users — pick one repository owned by an account on the list, walk five ordered phases (respond to every inbound comment, triage every unlabeled issue, review every open pull request not authored by the current login, file one bug report, submit one pull request), delegate each phase to its sub-skill, skip a phase silently when it finds no target, and stop after the pull request is submitted.
---

Read `CONVENTIONS.md` at the root of this plugin
  before doing anything else, hold every rule in that
  file as binding for the rest of the run, and pass it
  to every delegated sub-skill as the canonical source
  of voice, signature, tooling, and Markdown rules.

Read the input as a single list named in the user's
  prompt: a list of GitHub accounts in `<login>` form,
  where each entry is either an organization or a
  user; refuse to run when the list is empty or
  missing, and do not invent entries from memory.

Treat the account list as the set of GitHub accounts
  whose repositories this run is allowed to touch as
  a target; enumerate the public, non-archived,
  non-fork repositories owned by each account with
  `gh repo list <login> --limit 1000 --no-archived
  --source --json nameWithOwner --jq
  '.[].nameWithOwner'` and take the union as the
  repository set for the rest of the run.

Identify the current GitHub login once with
  `gh api user --jq .login` and capture it for the
  rest of the run, because every check that filters
  comments, pull requests, issues, or notifications
  by author or recipient depends on this value.

Ignore every issue whose `labels` array contains
  `wontfix`, `invalid`, or `duplicate` for the
  entire run: skip it when scanning notifications,
  exclude it from triage candidates, never pick it
  as a pull request target, and never comment,
  react, push a commit, or open a pull request
  against it.

Treat the submitted pull request as the ultimate
  goal of this run; every phase before Phase 5 is a
  preliminary contribution that runs first, and a
  preliminary phase that finds no target advances
  silently to the next phase rather than aborting
  the chain.

## Pick a single repository for the chain

Walk the derived repository set in the order the
  accounts were given — and, within each account, in
  the order `gh repo list` returned the repositories
  — and pick the first repository that carries at
  least one open issue labeled `help wanted`, `bug`,
  or `enhancement` that is unassigned and not
  authored by the current login, because Phase 5
  needs an actionable pull request target and that
  is the constraint the chain plans around.

Fall back to the first repository in the set when no
  repository carries an actionable pull request
  target, because Phases 1 through 4 still produce
  value even when the chain cannot reach Phase 5.

Operate on the chosen repository for every phase of
  the chain: every delegated sub-skill targets the
  same repository, and the chain never switches
  repositories mid-run.

## Phase 1: respond to every inbound comment

Fetch the unread notifications addressed to the
  current login with
  `gh api 'notifications?participating=true&all=false'
  --paginate`, and filter
  the list down to entries whose
  `repository.full_name` matches the chosen
  repository, whose `reason` is one of `mention`,
  `review_requested`, `comment`, or `author`, and
  whose `subject.type` is `Issue`, `PullRequest`,
  or `PullRequestReview`.

For each surviving notification, fetch the source
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

Discard a notification when the latest comment on
  the thread was authored by the current login, by
  the bot that opened the pull request, or by
  `github-actions[bot]` when the body is a pure CI
  status echo.

Discard a notification when the latest comment from
  another user has already been answered in-thread
  by the current login with either a reply or a
  referenced commit; an emoji reaction does not
  count as an answer.

Delegate to `respond-to-comment` once with the
  oldest surviving comment as input — passed as the
  artifact reference `<owner>/<repo>#<number>`
  together with the comment id — then wait for the
  sub-skill to finish, re-run the same notification
  scan and the same filters against the fresh
  GitHub state, and delegate again for the next
  oldest surviving comment, until no comment
  survives the filters.

Advance to Phase 2 when no comment survives the
  filters at the start of the phase or after the
  last delegation, because a quiet inbox is a valid
  outcome and the chain continues.

## Phase 2: triage every unlabeled issue

Fetch the open issues with
  `gh issue list --repo <owner>/<repo> --state open
  --limit 200 --json
  number,title,labels,assignees,createdAt`, and
  select every issue whose `labels` array is empty,
  because an unlabeled issue is invisible to triage
  and a single label is the cheapest contribution
  available on this pass.

Delegate to `triage-issue` once with the oldest
  unlabeled issue as input — passed as the artifact
  reference `<owner>/<repo>#<number>` — then wait
  for the sub-skill to finish, re-fetch the open
  issue list against the fresh GitHub state, and
  delegate again for the next oldest unlabeled
  issue, until no unlabeled issue remains.

Advance to Phase 3 when no unlabeled issue remains
  at the start of the phase or after the last
  delegation, because a fully-labeled backlog is a
  valid outcome and the chain continues.

## Phase 3: review every open pull request

Fetch the open pull requests with
  `gh pr list --repo <owner>/<repo> --state open
  --limit 200 --json
  number,title,author,reviews,headRefOid,createdAt`,
  and select every pull request not authored by the
  current login, not authored by a bot, and not
  already reviewed by the current login on its
  current head commit.

Treat a pull request as bot-authored when the
  `author.is_bot` field returned by `gh pr list --json
  author` is `true`, when the `author.login` ends in
  `[bot]` or starts with `app/`, when the `author.type`
  field is `Bot` on a direct REST API call, or when the
  login matches a known automation account such as
  `renovate[bot]`, `dependabot[bot]`,
  `github-actions[bot]`, `pre-commit-ci[bot]`,
  `mergify[bot]`, or `greenkeeper[bot]`.

Delegate to `review-pull-request` once with the
  oldest reviewable pull request as input — passed
  as the artifact reference `<owner>/<repo>#<number>`
  — then wait for the sub-skill to finish, re-fetch
  the pull request list against the fresh GitHub
  state, and delegate again for the next oldest
  reviewable pull request, until no reviewable pull
  request remains.

Advance to Phase 4 when no reviewable pull request
  remains at the start of the phase or after the
  last delegation, because an empty review queue is
  a valid outcome and the chain continues.

## Phase 4: file one bug report

Delegate to `file-bug-report` once with the chosen
  repository slug as input, and let the sub-skill
  pick the defect itself by reading the source
  tree; do not pass a defect, do not run the build,
  and do not pre-screen findings — the sub-skill
  owns the selection.

Advance to Phase 5 after the sub-skill finishes,
  whether the sub-skill filed a new issue or
  reported no defect worth filing, because a clean
  source tree is a valid outcome and the chain
  continues.

## Phase 5: submit one pull request

Pick a single pull request target inside the chosen
  repository before delegating: the oldest open
  issue labeled `help wanted` that is unassigned,
  not authored by the current login, and has no
  claimed work in the thread; fall back to the
  oldest open issue labeled `bug` or `enhancement`
  under the same conditions only when no
  `help wanted` candidate remains.

Skip every open issue authored by the current login
  when picking the pull request target, and fall
  through to the next candidate, because a bug
  reporter must not be the person who solves the
  bug — the fix needs a second pair of eyes, and a
  self-fixed report defeats the review value of
  filing it in the first place.

Delegate to `submit-pull-request` once with the
  picked target as input — passed as the artifact
  reference `<owner>/<repo>#<number>` together with
  the repository slug — and let the sub-skill
  enforce its own build, rebase, push, and CI
  contract.

Stop the run after `submit-pull-request` finishes,
  whether the sub-skill submitted a pull request or
  reported no actionable target: do not retry the
  phase, do not pick a second repository, do not
  start a second chain on the same pass, and re-run
  this skill from the top for the next pass.

## Sub-skill delegation rules

Pass the canonical conventions from `CONVENTIONS.md`
  to every delegated sub-skill, so the sub-skill's
  voice, signature, and tooling decisions follow the
  same rules across the chain.

Delegate to one sub-skill at a time and wait for it
  to finish before starting the next delegation,
  because each sub-skill writes to GitHub and the
  chain cannot interleave its writes safely.

Let each delegated sub-skill enforce its own
  contract — duplicate checks, build steps, comment
  style, stop conditions — without re-implementing
  those rules in this skill.

Treat a sub-skill that reports no actionable target
  as a silent skip: advance to the next phase, do
  not retry the same sub-skill inside the same
  phase, and do not abort the chain.

## Report

Report at the end of the run a short factual
  summary per phase: for each delegated sub-skill,
  the artifact URL it produced (the reply, the
  labeled or closed issue, the review, the new bug
  issue, or the new pull request); for each skipped
  phase, the reason for the skip (empty inbox,
  fully-labeled backlog, empty review queue, no
  defect worth filing, or no pull request target);
  and for the chain as a whole, the chosen
  repository and the final state of the ultimate
  goal (pull request submitted, or no pull request
  target found).
