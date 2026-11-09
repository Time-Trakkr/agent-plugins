# @timetrakkr/cli

The command-line half of the [Time Trakkr](https://timetrakkr.com) plugins for
Claude Code, Codex and Cursor. The plugins call it from the agent's hooks to
measure how long the agent worked in a repository your workspace has
connected, and it reports that time to Time Trakkr, where it waits on your
timesheet for you to review. At the end of the day it becomes a time entry on
its own if your workspace accepts automatically, which it does unless an admin
has turned that off. Time mapped by a `.timetrakkr.json` file in the
repository is never accepted automatically.

On its own, this package records nothing: time is measured by the hooks a
plugin installs, or, for Claude Code cloud sessions, by the hooks
`init --cloud` writes to a repository. Install a plugin to track time on your
own computer; use this package to sign in, check on things from a terminal, or
set up the Cursor CLI and Claude Code cloud sessions.

The same file, at the same version, is bundled in every plugin, so this
package and the plugins never disagree. Both keep their state in
`~/.timetrakkr`, so signing in with one signs in the other.

It needs Node 20 or later and has no dependencies.

## Install the plugin

The plugins are published at
[github.com/Time-Trakkr/agent-plugins](https://github.com/Time-Trakkr/agent-plugins), with the install command for
each host. For Claude Code:

```sh
claude plugin marketplace add Time-Trakkr/agent-plugins && claude plugin install timetrakkr@timetrakkr
```

## Sign in

```sh
npx @timetrakkr/cli login
```

Your browser opens on Time Trakkr. Sign in, pick your workspace and press
Allow. The workspace must have the Agent time add-on switched on. For a
workspace on a custom domain or on staging, add `--url <address>` once; the
address is remembered.

The sign-in can report agent time and read the workspace's repository rules,
and nothing else: it cannot read or change your time entries, reports,
invoices or anything else in your workspace. It is kept in an owner-only file
under `~/.timetrakkr`, renews itself, and lapses after 30 days without use. It
is listed under Integrations → Connected apps in Time Trakkr as "Time Trakkr CLI on" and this
computer's hostname, which is the one thing about the machine that signing in
sends. `npx @timetrakkr/cli logout` revokes it.

On a machine with no browser, set `TIMETRAKKR_TOKEN` to a personal access
token created with **Agent time only** under Integrations → Personal access
tokens, instead of signing in.

## Commands

| Command | What it does |
| --- | --- |
| `timetrakkr login` | Sign in through your browser, picking the workspace there |
| `timetrakkr logout` | Revoke the sign-in and delete it from this machine |
| `timetrakkr status` | What this directory is tracked to and why, and whether sign-in works |
| `timetrakkr off` / `on` | Stop or resume tracking the current agent session |
| `timetrakkr estimate --hours <n> --label "<text>"` | Record how long the agent said the work would take a person; shown beside the agent's time, never billed |
| `timetrakkr cursor setup [--yes] [--undo]` | Let the Cursor CLI report whole turns: adds two entries that do nothing themselves to `~/.cursor/hooks.json`, after showing them and asking. `--undo` removes exactly those entries |
| `timetrakkr init --cloud [--pin <version>] [--url <address>] [--yes] [--undo]` | Measure Claude Code cloud sessions in this repository: writes hooks to `.claude/settings.json` that run only in a cloud session, after showing them and asking. Commit the file. `--undo` removes exactly those hooks |
| `timetrakkr --version` | Print the version |

With `npx`, write `npx @timetrakkr/cli` in place of `timetrakkr`.

## Claude Code cloud sessions

A Claude Code cloud session does not load plugins. To measure its time:

1. Create a personal access token under Integrations → Personal access tokens,
   with **Agent time only** and **For a cloud environment** ticked.
2. In Claude Code on the web, edit the environment the repository uses and add
   a network secret for your workspace's address: header `Authorization`,
   prefix `Bearer`, and the token as the value. The token never enters the
   session.
3. In the repository, run `npx @timetrakkr/cli init --cloud` (add
   `--url <address>` for a custom domain) and commit `.claude/settings.json`.
4. Start a new session, on the environment that has the network secret.

The hooks do nothing outside a Claude Code cloud session, so nobody with the
plugin is counted twice. Cursor and Codex cloud agents check in instead; see
[agent time for agents in the cloud](https://timetrakkr.com/help/integrations/agent-time-cloud).

## What is sent, and what never is

Only for a repository your workspace has mapped to a project, by an admin's
rule or a `.timetrakkr.json` file in the repository. For any other repository
nothing about it is sent, not even its address.

Sent, once per turn: the session id, the day, the active seconds and number of
turns that day, when the first turn began and the last ended, the project and
task and the rule that chose them, the repository's address as
`host/owner/repo`, the branch, the model, token counts per model, and an
estimate if one was recorded.

Never sent: your prompts, the agent's replies, tool input or output, file
contents or file names, and diffs. The agent's transcript is read on your
machine for token counts, the model name and reply ids only.

The time between turns, while you read or are away, is not counted.

## Source

This package is built from the Time Trakkr repository and published with the
plugins, from [github.com/Time-Trakkr/agent-plugins](https://github.com/Time-Trakkr/agent-plugins), where each
release holds the exact file published here.

## License

MIT. See `LICENSE`.
