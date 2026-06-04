# Zealot

[![License](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/zerocracy/zealot/blob/master/LICENSE.txt)

Zealot is a bundle of disciplined skills for the everyday GitHub
  workflow, intended to run inside [Claude Routines].

The single top-level entry point is `make-one-contribution`:
  it picks one repository from a list of GitHub accounts and
  walks a fixed chain of preliminary contributions —
  responds to every inbound comment, triages every unlabeled
  issue, reviews every open pull request not authored by the
  current login, files one bug report — before ending on the
  ultimate goal of one submitted pull request.

Two installation paths are supported.

For a Claude Routine, reference the repository inside the routine prompt
  and invoke `make-one-contribution` against the list of target GitHub accounts,
  for example:

```text
Use this OAuth GitHub token: ghp_JI...tU.
Use this GitHub repository to learn new skills: zerocracy/zealot.
Use this list of GitHub accounts as the target: yegor256, jcabi, objectionary.
Use the make-one-contribution skill to make one contribution.
```

For Claude Code on a developer machine, install through the marketplace:

```text
/plugin marketplace add zerocracy/zealot
/plugin install zealot@zealot
```

Configure the routine to run every day,
  and you get a disciplined contributor to your GitHub repositories.

[Claude Routines]: https://code.claude.com/docs/en/routines
