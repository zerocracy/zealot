---
name: make-git-commit
description: |
  Use this skill when creating a Git commit. Follow the
  Conventional Commits 1.0.0 specification, write a short
  imperative subject, keep the body concise and factual,
  and never add a co-author trailer for Claude, Claude Code,
  or any other agent.
---

Run `git status` and `git diff --staged` before composing
  the message, so the subject and body describe what is
  actually staged and not what was planned.

Stage files explicitly with `git add <path>`; do not run
  `git add -A`, `git add .`, or `git commit -a`, because
  those sweep in unrelated changes and leak secrets.

Format the subject line as `<type>(<scope>): <description>`
  per the Conventional Commits 1.0.0 spec at
  https://www.conventionalcommits.org/en/v1.0.0/, where the
  scope in parentheses is optional and the colon-space
  separator is mandatory.

Prepend the related ticket number with a leading hash sign
  to the description (`fix(parser): #123 handle empty
  input`) whenever the commit is tied to a known issue or
  pull request, so the subject links the change to its
  tracking ticket; omit the prefix only when no ticket
  applies.

Pick the type from the standard set: `feat` for a new
  feature, `fix` for a bug fix, `docs`, `style`, `refactor`,
  `perf`, `test`, `build`, `ci`, `chore`, or `revert`; do
  not invent new types.

Append `!` after the type or scope (`feat!:` or
  `feat(api)!:`) and add a `BREAKING CHANGE:` footer when
  the commit introduces an incompatible change, so tooling
  can detect the break from the subject alone.

Write the description in the imperative mood, lowercase,
  with no trailing period, under 72 characters
  (`fix(parser): handle empty input`, not
  `Fixed the parser.`).

Leave one blank line between the subject and the body, and
  another blank line between the body and any footer; the
  parser relies on these blank lines to split the message.

Keep the body to a few short sentences that explain the
  motivation and the visible effect, wrap lines at 72
  columns, and omit the body entirely when the subject
  already says everything.

Do not add a `Refs:` footer when the subject already
  carries the `#<number>` prefix, because the reference is
  already visible and a second mention is noise; add a
  `Closes #<number>` or `Fixes #<number>` footer only when
  the host needs that exact token to auto-close the ticket
  on merge.

Never add a `Co-authored-by:` trailer naming Claude,
  Claude Code, Anthropic, or any other coding agent; the
  commit is authored by the human running the agent and
  attribution to a tool is misleading.

Never override the author or committer on the command line
  with `--author`, `-c user.name=`, `-c user.email=`,
  `GIT_AUTHOR_NAME`, or `GIT_COMMITTER_EMAIL`; rely on the
  defaults configured in the repository or the user's
  global Git config so the commit is attributed correctly.

Never add promotional trailers, signatures, emoji, or
  "Generated with" lines to the message; the commit log is
  a technical record, not a credit screen.

Do not pad the message with restated diffs, file lists, or
  obvious summaries (`updated foo.js to add a function`);
  the diff already shows that — the message must add the
  reason.

Do not bundle unrelated changes into one commit; split by
  intent so each commit has one type, one scope, and one
  reviewable purpose.

Do not pass `--no-verify`, `--no-gpg-sign`, or any flag
  that bypasses hooks or signing unless the user has
  explicitly asked for it; a failing hook means the commit
  is not ready.

Pass the message via a HEREDOC to `git commit -F -` or
  `git commit -m "$(cat <<'EOF' ... EOF)"` so blank lines,
  backticks, and quotes survive the shell intact.

After the commit, run `git log -1 --stat` to confirm the
  subject, body, footers, and file set match the intent;
  if anything is wrong, create a new commit rather than
  amending the previous one.
