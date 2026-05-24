---
name: file-bug-report
description: |
  Use this skill to file a single bug as a new GitHub
  issue against a specific repository.
---

Target the GitHub repository the user named.
Clone or pull the default branch before reading any code.
Verify the symptom against the source code when the user named a defect.
Walk the source tree breadth-first when the user did not name a defect.
Build a mental map of the main components by following imports from the entry points named in the README.
Do not run the build, tests, linters, or static analysis.
Do not modify files, create branches, or open pull requests.
Check the open issues for a duplicate before filing.
Discard the candidate when it matches an already-open issue.
Check the closed issues for the same symptom.
Look for a logical flaw, an inconsistency, or a defect when picking a bug from scratch.
Treat a missing useful feature as a valid bug, with a concrete pointer to the file or module where the new code would belong.
Treat repository-level health issues as valid bugs, like broken CI, missing or contradictory architecture docs, missing or insufficient linters and analyzers, noisy CI output, outdated dependencies, garbage files, non-standard layout, missing or low test coverage, or typos in code or docs.
Treat a build that is not strict enough as a valid bug, like warnings not promoted to errors, lint rules set to warn, partial checker enablement, permissive type checking, absent coverage thresholds, or CI steps swallowing non-zero exit codes.
Treat design inconsistencies as valid bugs only when the symptom can be tied to specific files, directories, or lines.
Rank candidates by severity before picking one.
File the most severe finding first.
Dig until a severe bug surfaces rather than stopping at the first surface-level finding.
Pick the single most concrete and verifiable finding from the highest-severity candidates.
Prefer a bug grounded in a specific file and line range over a vague architectural complaint.
Discard findings that are matters of taste or stylistic preference.
Discard findings that depend on guessing the runtime behavior or on assumptions the source does not justify.
Use a short, declarative title naming the symptom and the location, not a vague phrase like `Bug in parser`.
Write the body as a few short paragraphs covering the bug, why it is wrong, and a proposed fix.
Keep the body compact.
Include a file path and an approximate line number for the offending code.
Suggest a concrete fix in one or two sentences.
Do not propose refactors, rewrites, or sweeping redesigns.
Do not attach patches, diffs, or pull requests to the issue.
Do not invent reproduction steps, fabricate stack traces, or claim to have run the program.
The owner is the slug owner, or the top recent committer for an organization.
Post one follow-up comment `@`-mentioning the owner and offering to clarify.
Keep the comment to one or two sentences, ping one account, and never request a deadline.
Stop after the follow-up comment.
Stop without posting anything when no bug is found.
