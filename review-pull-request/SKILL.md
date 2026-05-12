---
name: review-pull-request
description: |
  Use this skill to review a single pull request on
  GitHub: read the diff and the surrounding code, judge
  each change against the project's conventions, leave
  inline comments where the code is wrong or unclear,
  and submit one review verdict — APPROVE, COMMENT, or
  REQUEST_CHANGES — with a short summary. One pull
  request per run, one verdict per run — then stop.
---

Treat the input as a single pull request reference —
  the PR — supplied by the user as an URL, an
  `owner/repo#number` triplet, or a number against the
  current repository; refuse to run if no pull request
  is given, and do not guess one from memory.

Run `git status` first; if the working directory has
  any uncommitted changes, untracked files, or staged
  hunks, stop immediately and do not continue, because
  this skill checks out the pull request branch and a
  dirty tree blocks the checkout.

Read the `README.md` and `CLAUDE.md` files at the root
  of the repository before reading the diff; they name
  the build command, the coding conventions, and the
  project-specific rules every inline comment and the
  final verdict in this run must respect.

Use the `gh` CLI for every read and write against
  GitHub, so authentication, rate limits, and pagination
  are handled by the official tool rather than by ad hoc
  HTTP calls.

Refuse to review a pull request authored by the same
  account that is running this skill; a self-review is a
  conflict of interest, and the user can read their own
  diff without a second pass.

Refuse to review a pull request that is already merged
  or closed; this skill leaves a verdict on an open
  pull request, and a post-merge comment cannot block a
  bad change.

Check out the pull request branch locally with
  `gh pr checkout <number>` so the diff can be read in
  context, the build can be run, and inline comments can
  point to the right file and line.

Fetch the metadata of the pull request — title, body,
  base branch, head branch, author, linked issues,
  reviewers, labels — with `gh pr view <number> --json`,
  so the review is grounded in the stated intent and not
  in a guess from the diff alone.

Read the full diff with `gh pr diff <number>` once to
  build a mental map of the change, then re-read each
  changed file in full from the working tree, because a
  diff hides the surrounding context that decides
  whether a hunk is correct.

Collect the existing review comments and review-summary
  bodies on the pull request
  (`gh api repos/<owner>/<repo>/pulls/<number>/comments`
  and
  `gh api repos/<owner>/<repo>/pulls/<number>/reviews`)
  so the new review does not duplicate feedback already
  posted by another reviewer.

Check the current CI status with `gh pr checks <number>`
  before judging the code; a red CI on a pull request
  often points to the same defect the diff hides, and
  the review must take it into account.

Judge each hunk against four questions in order: does
  it match the stated intent in the title and body, does
  it respect the conventions written in `README.md` and
  `CLAUDE.md`, does it preserve the existing invariants
  of the surrounding code, and does it cover its own
  edge cases with tests; raise an inline comment for
  every `no` answer.

Look for the common defects first: an off-by-one, a
  wrong branch, a misnamed variable that changes
  behavior, an unguarded null, a leaked resource, a
  missing test for a new code path, a missing update to
  a doc that the code now contradicts, a public method
  added without an interface declaration, an
  introspection or cast where polymorphism would do.

Look for scope creep: a refactor smuggled into a bug
  fix, a rename of unrelated identifiers, a reformatting
  of files not touched by the stated change, a new
  dependency that the body does not justify — and call
  it out inline, because scope creep multiplies the
  review surface and hides the real change.

Leave one inline comment per defect, anchored to the
  exact file and line of the offending code, and never
  bundle several defects into a single comment, because
  one comment per thread lets the author answer them
  one at a time with one commit each.

Write each inline comment as a few short sentences of
  plain prose: one sentence naming the defect, one
  sentence explaining why it is wrong (citing the
  project rule, the invariant, or the broken contract),
  and one sentence suggesting the smallest change that
  resolves it.

Use GitHub suggested-change blocks
  (` ```suggestion ... ``` `) only for a one-line patch
  that the author can apply with a single click;
  larger changes belong in plain prose, because a long
  suggestion block hides the reasoning behind the
  patch.

Keep every comment free of apology, flattery, or
  hedging — no `thanks`, no `nice work`, no `nit:`, no
  `IMO`, no `you might want to`, no emoji; the author
  needs the verdict, not the ceremony.

Do not overuse Markdown: no headings, no nested bullet
  lists, no tables, no collapsed sections, no banners —
  use plain prose in short paragraphs and reserve fenced
  code blocks for the rare case where quoting code is
  essential.

Run the build — every test, every linter, every static
  check named in `README.md` or `CLAUDE.md` — locally
  on the checked-out branch when the project's rules
  expect it, so the review reports whether the build
  passes on a clean machine and not only on the
  author's.

Decide one verdict — APPROVE, COMMENT, or
  REQUEST_CHANGES — and never invent a fourth.

Choose APPROVE when no inline comment names a real
  defect, the diff respects the project conventions,
  the tests cover the new behavior, and CI is green;
  approval is a positive claim that the change is
  ready to merge, not a default for a quiet review.

Choose REQUEST_CHANGES when at least one inline comment
  names a correctness, security, or contract defect
  that must be fixed before merge, or when CI is red
  on a failure the diff causes; a request-changes
  verdict blocks the merge and asks for a follow-up.

Choose COMMENT when the inline comments name only
  stylistic or scope concerns, the author can take or
  leave them, and the change is not so wrong that it
  must be blocked; a comment verdict leaves the merge
  decision to the author and the maintainer.

Compose the review-summary body as one short paragraph
  that states the verdict, names the strongest reason,
  and points to the inline comments by count — not as
  a duplicated list of every inline comment, because
  the inline comments already exist and a second copy
  is noise.

Submit the review in one call with
  `gh pr review <number> --approve|--request-changes|--comment --body-file -`
  or the equivalent
  `gh api -X POST repos/<owner>/<repo>/pulls/<number>/reviews`
  endpoint, so the inline comments and the verdict
  land as a single review and not as a stream of
  separate posts.

Never name Claude, Claude Code, Anthropic, an AI agent,
  a language model, or any other automation tool in the
  inline comments, the review-summary body, or any
  follow-up reply during this run — omit every
  signature, footer, banner, trailer, and disclosure
  that hints the content was machine-generated, so each
  post reads as if a human reviewer wrote it directly.

Do not merge the pull request, do not auto-merge it,
  do not close it, do not push commits to its branch,
  do not assign reviewers, do not change labels, and
  do not set milestones; the only write actions
  permitted are the inline comments and the single
  review verdict.

Do not open new issues or new pull requests as part
  of this skill; if the diff reveals a problem larger
  than the pull request, name it in the review-summary
  body and ask the author to file a follow-up issue,
  but do not file it on their behalf.

Stop the run after the review is submitted: do not
  pick a second pull request, do not revisit the same
  pull request after the author replies, and re-run
  this skill from the top for the next review.

Report at the end of the run a short factual summary:
  the pull request URL, the verdict, the number of
  inline comments, and the CI status at the time of
  review, so the user can scan the result without
  re-reading the whole thread.
