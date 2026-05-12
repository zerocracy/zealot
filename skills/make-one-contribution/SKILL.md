---
name: make-one-contribution
description: |
  Use this skill as the top-level entry point for a daily
  contribution run against a single GitHub repository:
  inspect the repository's open issues, its unlabeled
  issues, and its open pull requests, decide which one of
  four contribution types the repository needs most —
  file a bug report, classify an unlabeled issue, fix one
  issue, or review a pull request — pick a single target
  inside that type, delegate to the matching sub-skill,
  and stop. One contribution per run, one sub-skill per
  run — then stop.
---

Treat the input as a single GitHub repository in the
  form `<owner>/<repo>`; refuse to run if no repository
  is given, and do not invent one from memory.

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

Choose `file-bug-report` when the backlog is short —
  ten or fewer open issues — because a short backlog
  means the repository can absorb a fresh bug report
  without drowning the maintainer in untriaged noise.

Choose `classify-bug-report` when at least one open
  issue carries no labels at all, because an unlabeled
  issue is invisible to triage and a single label is
  the cheapest contribution available on this pass.

Choose `fix-one-issue` when the open pull request queue
  is short — five or fewer open pull requests — because
  a short queue means a new fix will be reviewed and
  merged rather than buried under older pull requests.

Choose `review-pull-request` otherwise, because a long
  pull request queue is the strongest signal that the
  repository needs reviewers more than it needs new
  reports, fixes, or labels.

Evaluate the four conditions in the order listed above
  and pick the first one that matches; do not score the
  conditions, do not combine them, and do not weight
  them — the order is the decision.

Pick a single target inside the chosen contribution
  type before delegating: for `file-bug-report`, no
  target is needed because the sub-skill picks the
  defect itself; for `classify-bug-report`, pick the
  oldest unlabeled open issue; for `fix-one-issue`,
  pick the oldest open issue labeled `bug` or
  `enhancement` that is unassigned and has no claimed
  work in the thread; for `review-pull-request`, pick
  the oldest open pull request not authored by the
  account running this skill.

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

Stop after the delegated sub-skill finishes: do not
  pick a second repository, do not pick a second
  contribution type, do not start a fix, a review, a
  classification, or a report on top of the first, and
  re-run this skill from the top for the next pass.

Report at the end of the run a short factual summary:
  the repository, the contribution type chosen, the
  reason this branch of the decision was taken (the
  backlog size, the unlabeled issue count, the pull
  request queue size), the target picked, and the URL
  of the artifact created or edited by the delegated
  sub-skill — the new issue, the labeled issue, the
  new pull request, or the review.
