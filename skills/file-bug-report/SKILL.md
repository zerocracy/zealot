---
name: file-bug-report
description: |
  Use this skill when the user wants to file a single bug as a
  new GitHub issue against a specific repository.
---

## Target

Target GitHub repository user named.
Clone or pull default branch before reading any code.

## Verification

Verify symptom against source code when user named defect.
Walk source tree breadth-first when user did not name defect.
Build mental map of main components.
Follow imports from entry points named in README.

## Restraint

Do not run build, tests, linters, or static analysis.
Do not modify files, create branches, or open pull requests.

## Duplicates

Check open issues for duplicate before filing.
Discard candidate when it matches already-open issue.
Check closed issues for same symptom.

## Selection

Look for logical flaw, inconsistency, or defect when picking bug.
Pick bug from scratch.
Treat missing useful feature as valid bug.
Give concrete pointer to file or module where new code belongs.

## Health

Treat repository-level health issues as valid bugs.
Count broken CI, missing or contradictory architecture docs.
Count missing or insufficient linters and analyzers.
Count noisy CI output, outdated dependencies, garbage files.
Count non-standard layout, missing or low test coverage.
Count typos in code or docs.

## Strictness

Treat build that is not strict enough as valid bug.
Count warnings not promoted to errors.
Count lint rules set to warn, partial checker enablement.
Count permissive type checking, absent coverage thresholds.
Count CI steps swallowing non-zero exit codes.

## Design

Treat design inconsistencies as valid bugs only when grounded.
Tie symptom to specific files, directories, or lines.

## Ranking

Rank candidates by severity before picking one.
File most severe finding first.
Dig until severe bug surfaces.
Do not stop at first surface-level finding.
Pick single most concrete and verifiable finding.
Pick from highest-severity candidates.

## Filtering

Prefer bug grounded in specific file and line range.
Prefer it over vague architectural complaint.
Discard findings that are matters of taste or stylistic preference.
Discard findings that depend on guessing runtime behavior.
Discard findings that rest on assumptions source does not justify.

## Title

Use short, declarative title naming symptom and location.
Do not use vague phrase like `Bug in parser`.

## Body

Write body as few short paragraphs.
Cover bug, why it is wrong, and proposed fix.
Keep body compact.
Include file path and approximate line number for offending code.
Suggest concrete fix in one or two sentences.

## Limits

Do not propose refactors, rewrites, or sweeping redesigns.
Do not attach patches, diffs, or pull requests to issue.
Do not invent reproduction steps or fabricate stack traces.
Do not claim to have run program.

## Owner

Owner is slug owner.
Owner is top recent committer for organization.
Post one follow-up comment `@`-mentioning owner and offering to clarify.
Keep comment to one or two sentences.
Ping one account, and never request deadline.

## Stopping

Stop after follow-up comment.
Stop without posting anything when you find no bug.
