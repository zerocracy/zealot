# Zealot

[![License](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/yegor256/zealot/blob/master/LICENSE.txt)

Zealot is a bundle of six disciplined skills for the everyday GitHub
  workflow, intended to run inside [Claude Routines]:

- `submit-pull-request` — open a single pull request the right way.
- `make-git-commit` — write a Conventional Commits message without padding.
- `address-reviewers-comments` — answer every review comment, one at a time.
- `file-bug-report` — read the code, find one defect, file it cleanly.
- `review-pull-request` — review a pull request without ceremony.
- `classify-bug-report` — label a fresh issue from the project's vocabulary.

This plugin is meant for routines only — there is no CLI install step, no
  `/plugin marketplace add`, no `/plugin install`; a routine pulls the
  skills from this repository directly at run time.

Reference the repository inside the routine prompt, for example:

```text
Use this OAuth GitHub token: ghp_JI...tU.
Use this GitHub repository to learn new skills: yegor256/zealot.
Use this GitHub repository as the target: yegor256/foo.
Use the file-bug-report skill to file one bug.
```

Configure the routine to run every day,
  and you get a disciplined contributor to your GitHub repositories.

[Claude Routines]: https://code.claude.com/docs/en/routines
