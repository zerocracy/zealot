---
name: address-reviewers-comments
description: |
  Use this skill to walk through a pull request and
  answer every comment left by a reviewer: list all
  review comments and review-summary comments, decide
  for each one whether to push back with a technical
  reason or to fix the code, apply the fix and commit
  it when the suggestion is right, and reply to the
  reporter on the same thread either way. One pull
  request per run, every comment answered exactly once
  — then stop.
argument-hint: "[owner/repo] [pr-number]"
user-invokable: false
---

Operate on the pull request `$1#$2`.

Run `git status` first; if the working directory has
  any uncommitted changes, untracked files, or staged
  hunks, stop immediately and do not continue, because
  this skill will commit and push during the run.

Check out the pull request branch locally (for example
  with `gh pr checkout <number>`), and run `git pull`
  on that branch so the work starts from the latest
  state of the head ref and not from a stale local
  copy.

Collect every comment on the pull request from three
  sources: inline review comments
  (`gh api repos/<owner>/<repo>/pulls/<number>/comments`),
  review-summary bodies
  (`gh api repos/<owner>/<repo>/pulls/<number>/reviews`),
  and issue-level comments on the pull request
  (`gh api repos/<owner>/<repo>/issues/<number>/comments`),
  because reviewers leave feedback in all three places
  and missing one source means missing comments.

Discard comments authored by the pull request author
  themselves, by the bot that opened the pull request,
  and by the `github-actions[bot]` account when the
  text is purely a CI status echo; every other comment
  counts as reviewer feedback and must be answered.

Treat suggestions from `copilot-pull-request-reviewer`,
  `coderabbitai`, `sonarcloud`, `codecov`, and similar
  automated reviewers as reviewer comments too, because
  the contract of this skill is to answer feedback
  regardless of whether the author is a human or a bot.

Skip a comment only when it has already been answered
  in-thread by the pull request author with either a
  reply or a referenced commit — a reply alone counts
  as answered, an emoji reaction does not, and a
  resolved-but-unreplied thread does not.

Build a working list of unanswered comments in the
  order GitHub returns them — oldest first — and walk
  the list one comment at a time, never in parallel,
  so every reply lands on the right thread and no
  comment is answered twice.

For each comment, read the surrounding code at the
  exact file and line the comment points to before
  deciding anything; a reply written without looking
  at the code is a guess, and guesses waste the
  reviewer's time.

Classify each comment into one of two outcomes —
  FIX or PUSH-BACK — and never invent a third; FIX
  means the reviewer is right and the code must
  change, PUSH-BACK means the reviewer is wrong,
  out of scope, or based on a misreading and the
  code must stay.

Choose FIX when the comment names a real defect, a
  violated convention, a missing test, a broken
  invariant, a typo, a security or correctness
  issue, or any concrete improvement that fits the
  scope of the current pull request.

Choose PUSH-BACK when the comment asks for work
  outside the scope of the pull request, contradicts
  the project conventions written in `README.md` or
  `CLAUDE.md`, restates a personal style preference
  with no technical basis, is based on a misreading
  of the diff, or duplicates feedback already
  addressed elsewhere in the same pull request.

When the right outcome is genuinely unclear after
  reading the code and the project rules, prefer
  FIX over PUSH-BACK, because applying a small change
  costs less than a debate and keeps the review
  moving.

For a FIX outcome, edit the source by hand, change
  only what the comment demands, and do not refactor
  neighboring code, rename APIs, or clean up unrelated
  style under cover of the same edit.

Run the full build — every test, every linter, every
  static check — after each FIX, and stabilize it
  before committing: every previously-passing test
  must still pass, every linter must be clean, and
  no new warning may appear.

Commit each FIX on its own with a message that
  references the comment (for example by quoting the
  reviewer login and the comment URL) and explains
  what changed; do not bundle several unrelated FIXes
  into one commit, because a one-to-one mapping
  between comment and commit lets the reviewer
  verify each answer in isolation.

Push the branch after each commit (or after a small
  batch of commits when the build is slow) so the
  reviewer sees progress and the CI runs against the
  latest state.

Reply to the comment on the same thread no matter
  the outcome: for a FIX, post a short reply that
  names the commit SHA and states what was changed;
  for a PUSH-BACK, post a short reply that states
  the technical reason the suggestion does not apply,
  citing the project rule, the scope boundary, or
  the misreading.

Use the GitHub reply endpoints to keep each reply on
  the original thread — for inline review comments
  use
  `gh api -X POST repos/<owner>/<repo>/pulls/<number>/comments/<comment_id>/replies`,
  for review-summary feedback reply with a new
  review or an issue comment that quotes the
  original — and never bundle replies into a single
  generic comment at the bottom of the pull request.

Keep every reply short and factual.

Do not mark a thread resolved without first posting
  a reply, and do not mark a thread resolved on
  behalf of the reviewer when the resolution is in
  dispute; let the reviewer close their own thread
  after a PUSH-BACK.

If a FIX triggers CI again, watch the checks (for
  example with `gh pr checks --watch`) and do not
  consider that comment answered until every check
  on the new commit is green; a red CI on a fix
  commit means the fix is incomplete.

If CI surfaces a failure unrelated to the current
  comment, fix it inside the same run rather than
  ignoring it, because a pull request that is
  green-on-merge is the contract of every fix in
  this skill.

Do not approve the pull request, do not request
  changes on it, do not merge it, do not rebase it,
  do not force-push, and do not close any thread
  the reviewer opened; the only write actions
  permitted are commits, pushes, and replies to
  existing comments.

Do not open new issues, new pull requests, or new
  branches as part of this skill; if a comment
  reveals a problem larger than the current pull
  request, write that down in the PUSH-BACK reply
  and ask the reviewer to file a follow-up issue,
  but do not file it on their behalf.

Recheck the working list after every reply and
  every push, because new reviewer comments may
  arrive while the run is in progress; pick up any
  new comment that lands on the pull request and
  treat it the same way as the original batch.

Stop the run only when every comment in the working
  list — including comments that arrived mid-run —
  has been answered with either a commit-and-reply
  pair or a PUSH-BACK reply, and every CI check on
  the latest commit is green.

Report at the end of the run a short factual summary:
  the number of comments answered with a FIX, the
  number answered with a PUSH-BACK, the list of
  commit SHAs pushed during the run, and the final
  CI status of the pull request, so the reviewer
  can scan the result without re-reading the whole
  thread.
