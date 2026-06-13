---
name: make-one-contribution
description: |
  Use this skill as the daily entry point for a
  contribution chain against a list of GitHub orgs and
  users, walking five ordered phases and delegating
  each to its sub-skill.
---

Read `CONVENTIONS.md` at the root of this plugin before doing anything else.
Hold every rule in `CONVENTIONS.md` as binding for the rest of the run.
Pass `CONVENTIONS.md` to every delegated sub-skill as the canonical source of voice, signature, tooling, and Markdown rules.
Read the input as a single list of GitHub accounts in `<login>` form.
Refuse to run when the list is empty or missing.
Do not invent entries from memory.
Enumerate the public, non-archived, non-fork repositories owned by each account, and take the union as the repository set for the rest of the run.
Identify the current GitHub login once and capture it for the run.
Ignore every issue whose `labels` array contains `wontfix`, `invalid`, or `duplicate` for the entire run.
Never comment, react, push a commit, or open a pull request against an ignored issue.
Treat the submitted pull request as the ultimate goal of the run.
Advance silently to the next phase when a preliminary phase finds no target.

## Pick a single repository for the chain

Pick a random repository carrying at least one open issue labeled `help wanted`, `bug`, or `enhancement` that is unassigned and not authored by the current login.
Fall back to a random repository in the set when no repository carries an actionable pull request target.
Operate on the chosen repository for every phase of the chain.

## Phase 1: respond to every inbound comment

Fetch the unread notifications addressed to the current login and filter them down to the chosen repository.
Keep only entries whose `reason` is `mention`, `review_requested`, `comment`, or `author`, and whose `subject.type` is `Issue`, `PullRequest`, or `PullRequestReview`.
For each surviving notification, fetch the source comment from the matching endpoint.
Locate the most recent comment authored by a login other than the current login that mentions the current login or sits on a thread the current login already participated in.
Discard a notification when the latest comment was authored by the current login, the bot that opened the pull request, or `github-actions[bot]` echoing CI status.
Discard a notification when the latest comment from another user has already been answered in-thread by the current login with a reply or a referenced commit.
Delegate to `respond-to-comment` once with the oldest surviving comment.
Wait for the sub-skill to finish, re-scan the notifications against fresh GitHub state, and delegate again for the next oldest surviving comment.
Repeat until no comment survives the filters.
Advance to Phase 2 when no comment survives the filters.

## Phase 2: triage every unlabeled issue

Fetch the open issues sorted by creation date.
Select every issue whose `labels` array is empty.
Delegate to `triage-issue` once with the oldest unlabeled issue.
Wait for the sub-skill to finish, re-fetch the open issue list, and delegate again for the next oldest unlabeled issue.
Repeat until no unlabeled issue remains.
Advance to Phase 3 when no unlabeled issue remains.

## Phase 3: review every open pull request

Fetch the open pull requests sorted by creation date.
Select every pull request not authored by the current login, not authored by a bot, and not already reviewed by the current login on its current head commit.
Treat a pull request as bot-authored when `author.is_bot` is `true`, the login ends in `[bot]` or starts with `app/`, the `author.type` is `Bot`, or the login matches a known automation account like `renovate[bot]`, `dependabot[bot]`, `github-actions[bot]`, `pre-commit-ci[bot]`, `mergify[bot]`, or `greenkeeper[bot]`.
Delegate to `review-pull-request` once with the oldest reviewable pull request.
Wait for the sub-skill to finish, re-fetch the pull request list, and delegate again for the next oldest reviewable pull request.
Repeat until no reviewable pull request remains.
Advance to Phase 4 when no reviewable pull request remains.

## Phase 4: file one bug report

Delegate to `file-bug-report` once with the chosen repository slug.
Let the sub-skill pick the defect itself by reading the source tree.
Do not pass a defect, do not run the build, and do not pre-screen findings.
Advance to Phase 5 after the sub-skill finishes.

## Phase 5: submit one pull request

Pick the oldest open issue labeled `help wanted` that is unassigned, not authored by the current login, and has no claimed work in the thread.
Fall back to the oldest open issue labeled `bug` or `enhancement` under the same conditions when no `help wanted` candidate remains.
Skip every open issue authored by the current login when picking the pull request target.
Prepare the working tree before delegating to `submit-pull-request`.
Clone the chosen repository or pull inside an existing clone.
Check out a feature branch off the default branch named after the picked target.
Implement the smallest fix that resolves the picked issue end to end.
Delegate to `make-git-commit` to commit the staged change.
Delegate to `submit-pull-request` once with the picked target as `<owner>/<repo>#<number>` and the prepared feature branch.
Stop the run after `submit-pull-request` finishes.
Re-run this skill from the top for the next pass.

## Sub-skill delegation rules

Pass the canonical conventions from `CONVENTIONS.md` to every delegated sub-skill.
Delegate to one sub-skill at a time and wait for it to finish before starting the next.
Let each delegated sub-skill enforce its own contract.
Treat a sub-skill that reports no actionable target as a silent skip.

## Report

Report a short factual summary per phase, including the artifact URL each sub-skill produced, the reason for any skip, and the chain's final state.
