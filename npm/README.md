# @timetrakkr/cli

The command-line half of the [Time Trakkr](https://timetrakkr.com) plugins for
Claude Code, Codex and Cursor. The plugins call it from the agent's hooks to
measure how long the agent worked in a repository your workspace has
connected, and it reports that time to Time Trakkr, where it waits on your
timesheet for you to review. Nothing becomes a time entry on its own.

On its own, this package records nothing: time is measured by the hooks a
plugin installs. Install a plugin to track time; use this package to sign in
or check on things from a terminal.

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
is listed under Connected apps in Time Trakkr as "Time Trakkr CLI on" and this
computer's hostname, which is the one thing about the machine that signing in
sends. `npx @timetrakkr/cli logout` revokes it.

On a machine with no browser, set `TIMETRAKKR_TOKEN` to a personal access
token created with **Agent time only** instead of signing in.

## Commands

| Command | What it does |
| --- | --- |
| `timetrakkr login` | Sign in through your browser, picking the workspace there |
| `timetrakkr logout` | Revoke the sign-in and delete it from this machine |
| `timetrakkr status` | What this directory is tracked to and why, and whether sign-in works |
| `timetrakkr off` / `on` | Stop or resume tracking the current agent session |
| `timetrakkr estimate --hours <n> --label "<text>"` | Record how long the agent said the work would take a person; shown beside the agent's time, never billed |
| `timetrakkr --version` | Print the version |

With `npx`, write `npx @timetrakkr/cli` in place of `timetrakkr`.

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
