---
name: file-bug-report
description: |
  Use this skill to file a single bug as a new GitHub
  issue against a specific repository — either a defect
  the user has already named, or one found by reading the
  source code with no builds and no tests. Confirm the
  bug is not a duplicate, write a concise report that
  names the symptom, points to the code, and suggests a
  fix, and post one short follow-up comment pinging the
  repository owner. One bug per run, one issue per run,
  one ping per run — then stop.
---

Treat the input as a single GitHub repository in the
  form `<owner>/<repo>` plus an optional description of
  the defect; refuse to run if no repository is given,
  and do not invent one from memory.

Clone or pull the repository's default branch into a
  local working directory before reading any code, so the
  analysis runs against the latest committed state and
  not a stale snapshot.

When the user already named the defect, confirm it
  against the source code: open the files the user named,
  read the surrounding context, and verify the symptom
  before writing the issue, because a report based on a
  misreading wastes the maintainer's time.

When the user did not name a defect, walk the source
  tree breadth-first: list the top directories, open the
  entry points named in the README, follow imports into
  the modules they touch, and build a mental map of the
  main components before zooming in.

Do not run the build, do not execute the test suite, do
  not start any linter, and do not invoke any static
  analysis tool — this skill is a read-only inspection of
  the source code.

Do not modify a single file in the repository, do not
  create branches, and do not open pull requests; the
  only writes this skill performs are the new issue and
  its follow-up comment on GitHub.

List every open issue in the repository (for example
  with
  `gh issue list --repo <owner>/<repo> --state open --limit 200 --json number,title,body,labels`)
  and skim each title and body so the candidate defect
  can be checked against the existing backlog.

Discard any candidate defect that matches an
  already-open issue by topic, file, symptom, or proposed
  fix — a duplicate report wastes the maintainer's time
  and signals careless triage.

Search the closed issues too for the same symptom (for
  example with
  `gh issue list --repo <owner>/<repo> --state closed --search <keywords>`),
  because a defect already debated and rejected,
  wontfixed, or marked as working-as-intended is not
  worth re-filing.

When picking a defect from scratch, look for exactly one
  of: a logical flaw (an off-by-one, a wrong branch, a
  misnamed variable that changes behavior), an
  inconsistency (two files contradicting each other, a
  doc lying about the code, a name used in two
  incompatible senses), or a defect (a crash path, a
  leaked resource, an unguarded null, a broken contract,
  a missing edge case).

Treat repository-level health issues as valid bug
  candidates too: one or more broken (red) CI jobs on
  `master`; a missing architecture section in the
  `README.md` that would explain the product; an
  architecture section that contradicts the actual code;
  missing or insufficient linters, static analyzers, or
  checkers wired into CI; noisy CI output that buries
  real signal under excess logs; outdated dependencies
  that can be upgraded; garbage files or unused
  directories left in the tree; a file layout that
  diverges from the de-facto standard for this type of
  product; missing or unpublished test coverage;
  published coverage below eighty percent; or typos and
  grammar mistakes in code, comments, or documentation.

Treat design inconsistencies as another class of bug —
  lack of modularity, inadequate error handling,
  inconsistent naming conventions, inadequate data
  modeling, overlooked testability, poor exception
  handling, overuse of global state, tight coupling to
  external services, inadequate input validation,
  insufficient documentation, lack of scalability, poor
  concurrency management, poor resource management,
  inconsistent coding styles, lack of code reuse,
  insufficient logging, missing design patterns where
  they would clearly apply, lack of automated testing,
  poor separation of concerns, poor dependency
  management, or overengineering — but only when the
  symptom can be tied to specific files, directories, or
  lines, and only when it is a demonstrable defect
  rather than a matter of taste.

Pick the single most concrete and verifiable finding
  from the candidates, and drop the rest — this skill
  files one bug per run, not a shortlist.

Prefer a bug grounded in a specific file and line range
  over a vague architectural complaint, because a
  maintainer can act on a pointer and cannot act on a
  feeling.

Discard findings that are matters of taste, stylistic
  preference, or "I would have written this differently"
  — those belong in a code review, not in a bug tracker.

Discard findings that depend on guessing the runtime
  behavior or on assumptions the source does not justify,
  because this skill commits to what the code actually
  says.

Open the new issue with
  `gh issue create --repo <owner>/<repo> --title ... --body ...`
  using a short, declarative title that names the symptom
  and the location — for example
  `Parser drops trailing newline in foo.go` — and not a
  vague phrase like `Bug in parser`.

Write the body as a few short paragraphs of plain prose:
  one paragraph naming the bug and the file/line, one
  paragraph explaining why it is wrong, and one paragraph
  proposing a concrete fix.

Keep the body compact — a handful of sentences, not a
  wall of text — because maintainers read short bug
  reports and skim long ones.

Include a file path and an approximate line number for
  the offending code (for example
  `src/parser/lexer.go:142`) so the maintainer can jump
  straight to it without searching.

Suggest a fix in one or two sentences that names the
  change in concrete terms — which condition to flip,
  which branch to remove, which guard to add, which name
  to align — and stop there.

Do not propose a refactor, a rewrite, or a sweeping
  redesign as the fix; the suggestion must be the
  smallest change that resolves the reported defect.

Do not attach a patch, a diff, or a pull request to the
  issue itself, because the contract of this skill ends
  at the bug report and its follow-up comment.

Do not invent reproduction steps the source code does
  not support, do not fabricate stack traces, and do not
  claim to have run the program — this is a
  static-reading bug report and must read like one.

Identify the repository owner as the `owner` of the slug
  when it is a user account, or as the most active recent
  committer to the default branch when the owner is an
  organization (for example via
  `gh api repos/<owner>/<repo>/contributors` combined
  with `gh api repos/<owner>/<repo>/commits?since=...`).

After the issue is created, capture the new issue number
  from the `gh issue create` output and use it for the
  follow-up comment in the next step.

Post one follow-up comment on the new issue (for example
  with
  `gh issue comment <number> --repo <owner>/<repo> --body ...`)
  that `@`-mentions the owner, asks them to take a look
  when they have a moment, and offers to clarify if
  anything in the report is unclear.

Keep the follow-up comment to one or two sentences of
  plain prose — no headings, no bullet lists, no emojis,
  no AI-disclosure boilerplate — because a ping is a
  ping, not a second report.

Do not ping more than one account in the follow-up
  comment, do not @-mention the whole organization, and
  do not request a deadline or a priority label.

Stop after the follow-up comment is posted: do not pick
  a second bug, do not file a second issue, do not open a
  pull request, and do not start a fix — re-run this
  skill from the top for the next bug.

If no bug is found, just stop: do not post anywhere, do
  not file a placeholder issue, and do not leave a
  follow-up comment.
