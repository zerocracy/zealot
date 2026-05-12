# Zealot

[![License](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/yegor256/zealot/blob/master/LICENSE.txt)

Zealot is a bundle of disciplined skills for the everyday GitHub
  workflow, intended to run inside [Claude Routines].

The single top-level entry point is `make-one-contribution`:
  it inspects the target repository, picks the one contribution type
  the repository needs most today — file a bug report, classify an
  unlabeled issue, fix one issue, or review a pull request —
  delegates to the matching sub-skill, and stops.

This plugin is meant for routines only — there is no CLI install step, no
  `/plugin marketplace add`, no `/plugin install`; a routine pulls the
  skills from this repository directly at run time.

Reference the repository inside the routine prompt and invoke
  `make-one-contribution` against the target repository, for example:

```text
Use this OAuth GitHub token: ghp_JI...tU.
Use this GitHub repository to learn new skills: yegor256/zealot.
Use this GitHub repository as the target: yegor256/foo.
Use the make-one-contribution skill to make one contribution.
```

Configure the routine to run every day,
  and you get a disciplined contributor to your GitHub repositories.

[Claude Routines]: https://code.claude.com/docs/en/routines
