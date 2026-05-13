---
name: submit-pull-request
description: |
  Use this skill to open a single pull request against a
  GitHub repository: confirm the working directory is
  clean and the branch is pushed, write a short
  declarative title, fill the body with motivation,
  visible effect, and a test plan grounded in what was
  actually run, and submit it with `gh pr create`. One
  pull request per run, one branch per run — then stop.
---

Treat the input as a single local branch on a single
  GitHub repository; refuse to run if the current
  directory is not a Git working tree, if the branch is
  the default branch (`main` or `master`), or if no
  upstream remote is configured.

Run `git status` first; if the working directory has any
  uncommitted changes, untracked files, or staged hunks,
  stop immediately and ask the user to commit, stash, or
  discard them, because every change in the diff must be
  recorded in a commit before the pull request opens.

Identify the base branch from
  `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`
  rather than guessing `main` or `master`, because the
  repository may use a different default.

Refuse to run on the base branch itself; this skill
  opens a pull request from a feature branch, and a pull
  request from the base branch into itself is a contract
  violation.

Refuse to run when an open pull request already exists
  for the current branch
  (`gh pr list --head <branch> --state open --json number`),
  because two pull requests against the same head split
  review across threads; ask the user to update the
  existing one instead.

Identify the linked issue, if any, from the branch name
  (typical patterns: `<number>-...`, `issue-<number>`,
  `fix-<number>`), from a `Closes #<number>` or
  `Fixes #<number>` token in the branch's commit
  messages, or from the input the user passed on the
  run; skip the next three checks when no linked issue
  surfaces, and apply them in full otherwise.

Stop the run immediately if the linked issue is already
  assigned to any GitHub account — read the `assignees`
  array with
  `gh issue view <number> --json assignees,author,comments`;
  an assigned issue is taken, and submitting a parallel
  pull request forks the effort and wastes review time.

Stop the run immediately if any comment on the linked
  issue announces intent to work on it — phrases such
  as `I will take this`, `working on it`, `I am on it`,
  `assigning to myself`, `let me handle this`, or any
  equivalent claim posted by an account other than the
  issue author — because the issue is already claimed
  by another contributor.

Stop the run immediately if the linked issue thread
  already carries a comment posted by the same GitHub
  account that runs this skill — resolve the login with
  `gh api user --jq .login` and compare it against the
  `user.login` field of every comment; a prior comment
  from this account means the issue has already been
  engaged on an earlier run, and a second submission
  would duplicate the work.

Fetch and rebase the branch onto the latest base
  (`git fetch origin && git rebase origin/<base>`) before
  opening the pull request, so the diff is computed
  against the current head of the base branch and merge
  conflicts surface locally instead of on GitHub.

Resolve every rebase conflict by hand, re-run the build
  after each conflict resolution, and never resort to
  `git rebase --skip` or `-X ours`/`-X theirs` shortcuts
  that drop work without review.

Push the branch with `git push -u origin <branch>` (or
  `git push --force-with-lease` after a rebase) and
  confirm the push succeeded before invoking
  `gh pr create`, because a pull request without a
  pushed head is a stub.

Run the full build — every test, every linter, every
  static check named in `README.md` or `CLAUDE.md` —
  before opening the pull request, and stabilize it; a
  red local build is not ready for review.

Compose the pull request title as a short, declarative
  sentence in the imperative mood, lowercase except for
  proper nouns, under 72 characters, and free of trailing
  punctuation
  (`fix parser drop of trailing newline in foo.go`, not
  `Fixed the parser.`).

Prepend the related ticket number with a leading hash
  sign to the title (`#123 handle empty parser input`)
  when the pull request closes or refers to an issue, so
  the title links the change to its tracking ticket; omit
  the prefix only when no ticket applies.

Match the title to the dominant commit type in the
  branch (`feat`, `fix`, `docs`, `refactor`, `perf`,
  `test`, `chore`) when the project's commit style
  expects it, so the merged commit follows the same
  convention as the branch.

Write the body as three short paragraphs of plain prose:
  one paragraph naming the motivation and the linked
  issue, one paragraph describing the visible effect of
  the change, and one paragraph listing the checks that
  were run and their outcome.

Add a `Closes #<number>` or `Fixes #<number>` line on
  its own when the pull request resolves an open issue
  and the host needs that exact token to auto-close the
  ticket on merge; omit the line otherwise.

Ground the test-plan paragraph in commands that actually
  ran on the local machine — name the command, the
  outcome, and any platform-specific caveat — and never
  paste a checklist of intentions that were not
  executed.

Keep the body compact: a few short sentences per
  paragraph, no walls of text, no exhaustive file lists,
  no restated diff, because reviewers skim the
  description and read the diff.

Pass the title and body to `gh pr create` via a HEREDOC
  to `--body-file -` or `--body "$(cat <<'EOF' ... EOF)"`,
  so blank lines, backticks, and quotes survive the
  shell intact.

Set the base branch explicitly with `--base <default>`
  and the head branch explicitly with `--head <branch>`,
  even when they match the defaults, so the pull request
  is not accidentally opened against the wrong target on
  forks or upstream remotes.

Mark the pull request as draft with `--draft` only when
  the user explicitly requested a draft, and never mark
  it ready-for-review automatically; the user owns that
  decision.

Do not request reviewers, do not assign labels, do not
  set milestones, and do not assign the pull request to
  anyone unless the user explicitly asked for those
  actions on the run.

Do not merge the pull request, do not approve it, do
  not auto-merge it, and do not force-push to it after
  it is open; the contract of this skill ends when
  `gh pr create` returns the URL.

Capture the pull request URL from the `gh pr create`
  output and run `gh pr checks <number> --watch` once,
  so a CI failure on the first build surfaces inside the
  same run rather than landing on the reviewer.

If the first CI run fails, fix the failure inside this
  run by committing and pushing additional commits to
  the same branch, and do not consider the pull request
  submitted until every required check is green or until
  the user explicitly accepts a red status.

Report at the end of the run a short factual summary:
  the pull request URL, the base and head branches, the
  list of commits pushed during the run, and the final
  CI status, so the user can scan the result without
  re-reading the whole thread.
