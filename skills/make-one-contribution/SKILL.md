---
name: make-one-contribution
description: |
  Use this skill as the top-level entry point for a daily
  contribution run against a single GitHub repository:
  first, check whether any pull request submitted earlier
  by the current GitHub user is still waiting on a reply
  to a reviewer comment, and if so delegate to
  `address-reviewers-comments` and stop; only when no
  such pull request exists, inspect the repository's open
  issues, its unlabeled issues, and its open pull
  requests, decide which one of four contribution types
  the repository needs most — file a bug report, classify
  an unlabeled issue, submit a pull request, or review a
  pull request — pick a single target inside that type,
  delegate to the matching sub-skill, and stop. One
  contribution per run, one sub-skill per run — then
  stop.
argument-hint: "[owner/repo]"
user-invokable: true
license: MIT
metadata:
  author: Yegor Bugayenko
  version: "0.1.0"
  category: SDLC
---

Operate on the GitHub repository `$1`; refuse to run
  if `$1` is empty, and do not invent one from memory.

Use the `gh` CLI for every read against GitHub, so
  authentication, rate limits, and pagination are
  handled by the official tool rather than by ad hoc
  HTTP calls.

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
  same three sources `address-reviewers-comments` uses
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

Choose `address-reviewers-comments` first when at least
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
  `address-reviewers-comments` always wins when it
  matches.

Pick a single target inside the chosen contribution
  type before delegating: for `address-reviewers-comments`,
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

Never name Claude, Claude Code, Anthropic, an AI
  agent, a language model, or any other automation
  tool in any text the delegated sub-skill posts to
  GitHub during this run — omit every signature,
  footer, banner, trailer, and disclosure that hints
  the content was machine-generated.

Do not sign any artifact the sub-skill produces —
  an issue, a pull request, a commit, a review, or a
  comment — with an AI-disclosure line, a marketing
  footer, a "generated by" banner, or any equivalent
  boilerplate; the artifact stands on its content.

Talk like a human in every text the delegated
  sub-skill writes — every commit message, every issue
  body, every comment, every reply, every review —
  state the content directly, do not pad with
  pleasantries, and do not open with a summary of the
  project itself.

Do not overuse Markdown in any text the sub-skill
  posts to GitHub: no headings, no nested bullet
  lists, no tables, no collapsed sections, no emojis,
  no banners — use plain prose in short paragraphs
  and reserve fenced code blocks for the rare case
  where quoting code is essential.

Keep every comment, reply, and message the sub-skill
  posts free of apology, flattery, or hedging — no
  `thanks`, no `nice work`, no `good catch`, no
  `you are right`, no `nit:`, no `IMO`, no `you might
  want to`, no `sorry`, no emoji; the reader needs the
  content, not the ceremony.

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
