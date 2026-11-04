# Time Trakkr plugins for coding agents

Records how long your coding agent worked in a repository your workspace has
connected, and reports it to [Time Trakkr](https://timetrakkr.com), where it
waits on your timesheet. Version 1.0.0.

This repository is generated. Do not edit it here: changes are made in the
Time Trakkr repository and published on release.

## Install

You need Node 20 or later on your `PATH`. Nothing else is installed.

**Claude Code**

```sh
claude plugin marketplace add Time-Trakkr/agent-plugins && claude plugin install timetrakkr@timetrakkr
```

Or inside a session: `/plugin marketplace add Time-Trakkr/agent-plugins`, then
`/plugin install timetrakkr@timetrakkr`.

**Codex**

```sh
codex plugin marketplace add Time-Trakkr/agent-plugins && codex plugin add timetrakkr@timetrakkr
```

Then start Codex and trust the plugin's hooks with `/hooks`: Codex does not run
a plugin's hooks until you have.

**Cursor (the `cursor-agent` CLI)**

```sh
cursor-agent plugin marketplace add https://github.com/Time-Trakkr/agent-plugins
```

Then install `timetrakkr` from `/plugin`. The Cursor editor has no way for one
person to add a marketplace from a repository; there it needs an admin to
import this repository as a team marketplace.

## Sign in

Start a session in a repository your workspace has connected. If this machine
is not signed in, your browser opens once to sign in: sign in to Time Trakkr,
pick your workspace, and press Allow. Tracking starts at your next prompt.

If you closed that window, sign in from inside the session:

- Claude Code: `/timetrakkr:login`
- Codex: ask it to use the `timetrakkr-login` skill, or run
  `~/.codex/plugins/cache/timetrakkr/timetrakkr/1.0.0/bin/timetrakkr login`
- Cursor: `/timetrakkr-login`

Your workspace on a custom domain or on staging: sign in with
`timetrakkr login --url <address>` once; the address is remembered.

## What is recorded and what is not

Each plugin's README says exactly what is recorded, what is sent and what
never is. In short: turn start and end times, the repository's remote and
branch for a connected repository, and token counts. Never your prompts, the
agent's replies, tool input or output, or file contents.
