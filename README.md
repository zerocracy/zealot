# Zealot

[![License](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/yegor256/zealot/blob/master/LICENSE.txt)

Zealot is a bundle of disciplined skills for the everyday GitHub
  workflow, intended to run inside [Claude Routines].

The single top-level entry point is `make-one-contribution`:
  it inspects the target repository, picks the one contribution type
  the repository needs most today — file a bug report, triage an
  open issue, submit a pull request, or review a pull
  request —
  delegates to the matching sub-skill, and stops.

A second user-callable skill, `report-to-telegram`,
  summarizes the work just finished in the current conversation
  into one short Markdown paragraph and posts it to a Telegram
  chat via the Telegram Bot API; it reads the bot token from
  the `TELEGRAM_TOKEN` environment variable and the chat id
  from `TELEGRAM_CHAT_ID`, opens the message with one emoji
  that visualizes the outcome, and uses Markdown only for links
  and inline fixed-width text.

Two installation paths are supported.

For a Claude Routine, reference the repository inside the routine prompt
  and invoke `make-one-contribution` against the target repository,
  for example:

```text
Use this OAuth GitHub token: ghp_JI...tU.
Use this GitHub repository to learn new skills: yegor256/zealot.
Use this GitHub repository as the target: yegor256/foo.
Use the make-one-contribution skill to make one contribution.
```

For Claude Code on a developer machine, install through the marketplace:

```text
/plugin marketplace add yegor256/zealot
/plugin install zealot@zealot
```

Configure the routine to run every day,
  and you get a disciplined contributor to your GitHub repositories.

[Claude Routines]: https://code.claude.com/docs/en/routines
