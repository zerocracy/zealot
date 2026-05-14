---
name: respond-to-comment
description: |
  Use this skill to answer a single comment addressed
  to the current GitHub login — a review comment on a
  pull request, a review-summary body, an issue comment
  on a pull request, or a comment on an issue — by
  deciding whether to push back with a technical
  argument or to do what the comment asks and change
  the code, then replying on the same thread either
  way. One comment per run, answered exactly once —
  then stop.
---

Operate on the comment named in the user's prompt as
  `<owner>/<repo>#<number>` together with a comment
  identifier (a review-comment id, a review id, or an
  issue-comment id); refuse to run when no comment is
  named.

Run `git status` first; if the working directory has
  any uncommitted changes, untracked files, or staged
  hunks, stop immediately and do not continue, because
  this skill may commit and push during the run.

Identify the current GitHub login once with
  `gh api user --jq .login` and capture it for the
  rest of the run, because every check that filters a
  comment by author or recipient depends on this
  value.

Fetch the target artifact with `gh pr view <number>
  --repo <owner>/<repo> --json ...` for a pull request
  or `gh issue view <number> --repo <owner>/<repo>
  --json ...` for an issue, so the run knows the head
  ref, the author, the labels, and the state before
  touching anything.

For a pull request comment, check out the pull request
  branch locally (for example with `gh pr checkout
  <number>`), and run `git pull` on that branch so the
  work starts from the latest state of the head ref
  and not from a stale local copy.

For an issue comment, stay on the default branch and
  do not check out anything else, because an issue
  carries no head ref and no working branch to update.

Fetch the comment in question and the surrounding
  thread from the matching endpoint — inline review
  comments
  (`gh api repos/<owner>/<repo>/pulls/<number>/comments`),
  review-summary bodies
  (`gh api repos/<owner>/<repo>/pulls/<number>/reviews`),
  or issue-level comments
  (`gh api repos/<owner>/<repo>/issues/<number>/comments`)
  — and locate the exact comment by its id, because
  replying to the wrong thread looks like ignoring the
  reporter.

Discard the run when the comment is authored by the
  pull request author themselves, by the issue author
  when answering on their own issue, by the bot that
  opened the pull request, or by `github-actions[bot]`
  when the body is a pure CI status echo; only a
  comment from another user qualifies as feedback to
  answer.

Treat suggestions from `copilot-pull-request-reviewer`,
  `coderabbitai`, `sonarcloud`, `codecov`, and similar
  automated reviewers as comments to answer too,
  because the contract of this skill is to answer
  feedback regardless of whether the author is a human
  or a bot.

Skip the run when the comment has already been
  answered in-thread by the current login with either
  a reply or a referenced commit — a reply alone
  counts as answered, an emoji reaction does not, and
  a resolved-but-unreplied thread does not.

Read the surrounding code at the exact file and line
  the comment points to before deciding anything when
  the comment is on a pull request; a reply written
  without looking at the code is a guess, and guesses
  waste the reporter's time.

Read the issue body, every prior comment on the
  thread, the linked pull requests, and the referenced
  files before deciding anything when the comment is
  on an issue; an answer that ignores prior context
  restarts a discussion that has already moved on.

Classify the comment into one of two outcomes — FIX
  or PUSH-BACK — and never invent a third; FIX means
  the reporter is right and the code (or the issue, or
  the pull request body) must change, PUSH-BACK means
  the reporter is wrong, out of scope, or based on a
  misreading and nothing changes.

Choose FIX when the comment names a real defect, a
  violated convention, a missing test, a broken
  invariant, a typo, a security or correctness issue,
  a missing piece of information the reporter is
  asking for, or any concrete improvement that fits
  the scope of the current pull request or issue.

Choose PUSH-BACK when the comment asks for work
  outside the scope of the artifact, contradicts the
  project conventions written in `README.md` or
  `CLAUDE.md`, restates a personal style preference
  with no technical basis, is based on a misreading
  of the diff or the issue, or duplicates feedback
  already addressed elsewhere on the same thread.

When the right outcome is genuinely unclear after
  reading the code, the issue, and the project rules,
  prefer FIX over PUSH-BACK, because applying a small
  change or writing a concrete answer costs less than
  a debate and keeps the conversation moving.

For a FIX outcome on a pull request, edit the source
  by hand, change only what the comment demands, and
  do not refactor neighboring code, rename APIs, or
  clean up unrelated style under cover of the same
  edit.

For a FIX outcome on an issue, gather the missing
  information the reporter asked for — a stack trace,
  a version number, a reproduction, a clarification —
  and post it on the same thread; do not silently
  edit the issue body to insert it.

Run the full build — every test, every linter, every
  static check — after each source edit, and stabilize
  it before committing: every previously-passing test
  must still pass, every linter must be clean, and no
  new warning may appear.

Commit each FIX on its own with a message that
  references the comment (for example by quoting the
  reporter login and the comment URL) and explains
  what changed; do not bundle several unrelated FIXes
  into one commit, because a one-to-one mapping
  between comment and commit lets the reporter verify
  the answer in isolation.

Push the branch after the commit so the reporter sees
  progress and CI runs against the latest state.

Reply to the comment on the same thread no matter the
  outcome: for a FIX on a pull request, post a short
  reply that names the commit SHA and states what was
  changed; for a FIX on an issue, post a short reply
  with the requested information or a link to the
  follow-up pull request; for a PUSH-BACK, post a
  short reply that states the technical reason the
  suggestion does not apply, citing the project rule,
  the scope boundary, or the misreading.

Use the GitHub reply endpoints to keep the reply on
  the original thread — for inline review comments
  use
  `gh api -X POST repos/<owner>/<repo>/pulls/<number>/comments/<comment_id>/replies`,
  for review-summary feedback reply with a new review
  or an issue comment that quotes the original, and
  for an issue comment reply with
  `gh issue comment <number> --repo <owner>/<repo>
  --body <text>` — and never bundle the reply into a
  generic comment elsewhere on the artifact.

Start every reply with `@<login>` where `<login>` is
  the GitHub login of the comment author, so the
  reporter is notified and the thread shows who the
  answer is directed at.

Keep every reply short and factual.

Ground every PUSH-BACK reply in a concrete argument:
  cite the line of `README.md` or `CLAUDE.md` that
  contradicts the suggestion, name the scope boundary
  the suggestion crosses, quote the part of the diff
  the suggestion misreads, or point at the prior
  comment that already covered the same ground — a
  bare "I disagree" does not count as an argument.

Do not mark a thread resolved without first posting a
  reply, and do not mark a thread resolved on behalf
  of the reporter when the resolution is in dispute;
  let the reporter close their own thread after a
  PUSH-BACK.

If a FIX triggers CI again, watch the checks (for
  example with `gh pr checks --watch`) and do not
  consider the comment answered until every check on
  the new commit is green; a red CI on a fix commit
  means the fix is incomplete.

If CI surfaces a failure unrelated to the current
  comment, fix it inside the same run rather than
  ignoring it, because a pull request that is
  green-on-merge is the contract of every fix in this
  skill.

Do not approve the pull request, do not request
  changes on it, do not merge it, do not rebase it,
  do not force-push, do not close the issue, do not
  reopen the issue, do not relabel it, and do not
  reassign it; the only write actions permitted are
  commits, pushes, and replies to the existing
  comment.

Do not open new issues, new pull requests, or new
  branches as part of this skill; if the comment
  reveals a problem larger than the current artifact,
  write that down in the reply and ask the reporter
  to file a follow-up issue, but do not file it on
  their behalf.

Stop the run after the single comment named in the
  prompt is answered with either a commit-and-reply
  pair or a PUSH-BACK reply, and — when the comment
  was on a pull request and triggered a fix — every
  CI check on the latest commit is green.

Report at the end of the run a short factual summary:
  the comment URL, the outcome (FIX or PUSH-BACK),
  the commit SHA pushed during the run when the
  outcome was FIX, the reply URL, and the final CI
  status when the artifact was a pull request, so the
  reporter can scan the result without re-reading the
  whole thread.
