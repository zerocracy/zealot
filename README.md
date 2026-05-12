# Zealot

[![License](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/yegor256/zealot/blob/master/LICENSE.txt)

Zealot is a [Claude Code] plugin that bundles six disciplined skills for
  the everyday GitHub workflow:

- `submit-pull-request` — open a single pull request the right way.
- `make-git-commit` — write a Conventional Commits message without padding.
- `address-reviewers-comments` — answer every review comment, one at a time.
- `file-bug-report` — read the code, find one defect, file it cleanly.
- `review-pull-request` — review a pull request without ceremony.
- `classify-bug-report` — label a fresh issue from the project's vocabulary.

Inside a Claude Code session, install it from this repository:

```text
/plugin marketplace add yegor256/zealot
/plugin install zealot@zealot
```

The first command registers this repository as a plugin marketplace;
  the second installs the `zealot` plugin from it, which exposes every
  skill at the repository root to your sessions automatically.

To update later, run `/plugin marketplace update zealot`;
  to remove, run `/plugin uninstall zealot@zealot`.

The skills also work well inside Claude Routines, for example:

```text
Use this OAuth GitHub token: ghp_JI...tU.
Use this GitHub repository as the target: yegor256/foo.
Use the file-bug-report skill to file one bug.
```

Configure the routine to run every day,
  and you get a disciplined contributor to your GitHub repositories.

[Claude Code]: https://code.claude.com/docs/en/skills
