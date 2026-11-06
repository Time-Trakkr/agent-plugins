# Time Trakkr for Cursor

Records how long Cursor's agent worked in a repository and reports it to Time
Trakkr, where it waits on your timesheet for you to review. Nothing becomes a
time entry on its own.

## Status

Cursor support is being verified. Until then, if a session records nothing,
check `~/.timetrakkr/hook.log` and run `timetrakkr status` in the repository.

The command-line tool behind this plugin is the same one the Claude Code and
Codex plugins use.

## What is measured

A turn starts when you submit a prompt (`beforeSubmitPrompt`) and ends when
the agent stops (`stop`), whether it completed, was aborted or hit an error.
Agent time is the sum of turns. The time between turns is not counted.

Time the agent spends waiting for you to approve something **is** counted on
Cursor: its hooks document no signal for it.

## Install

Published from a public marketplace repository, with the CLI bundled, so
there is nothing to build. In the `cursor-agent` CLI:

```sh
cursor-agent plugin marketplace add https://github.com/<owner>/<repo>
```

then install `timetrakkr` from `/plugin`. The Cursor editor has no way for
one person to add a marketplace from a repository: an admin imports the
repository as a team marketplace (Teams or Enterprise). `<owner>/<repo>` is
the public plugins repository; the organization that will hold it is being
set up, and `docs/AGENT_PLUGINS_PUBLISHING.md` says how it is published. None
of this has been run against a real Cursor.

## Install for development

The plugin runs the CLI as one file, `bin/timetrakkr`. That file is a build
artifact and is not checked in. From the repository root, once and again after
any change to `packages/agent-cli`:

```sh
pnpm --filter @timetrakkr/cli build:plugin
```

Then either start the CLI with the plugin directory:

```sh
cursor-agent --plugin-dir /path/to/time-tracking-app/plugins/cursor
```

or, for the editor, copy this directory to `~/.cursor/plugins/local/timetrakkr`
and reload. It needs Node 20 or later on your `PATH`. For `timetrakkr status`,
`off`, `on` and `estimate` from a terminal, put `timetrakkr` on your `PATH`:

```sh
ln -s "$PWD/plugins/cursor/bin/timetrakkr" ~/.local/bin/timetrakkr
```

## Sign in

**Usually there is nothing to do.** The first session on a machine that is not
signed in, in a repository that could be tracked, opens your browser to sign
in to Time Trakkr. Sign in, pick your workspace and press Allow; tracking
starts at your next prompt. There is no address to type: the sign-in goes
through `https://timetrakkr.com`, which lists every workspace you belong to.

Cursor has no way to show a line at session start, so the browser opens with
no word from the session; the page it opens says what is asking.

The browser opens on its own at most once a day per machine, never while a
sign-in is already waiting, and never after `timetrakkr logout` until you sign
in again. If you closed it, type `/timetrakkr-login` in the agent chat, which
runs the bundled CLI to open the browser again and prints the link, or run
`timetrakkr login` in a terminal. `/timetrakkr-status` says what the session
is tracked to.

From a terminal it is `timetrakkr login`. For a workspace on staging or on a
custom domain, sign in once with `timetrakkr login --url <address>`; the
address is remembered.

The sign-in is kept in an owner-only file under `~/.timetrakkr`, renews
itself, and lasts as long as it is used at least once every 30 days. It can
reach the two agent-time routes and nothing else in your workspace.
`timetrakkr logout` revokes it and deletes it from this machine; you can also
remove it under Developer → Connected apps.

Signing in sends this computer's hostname to the server, as part of the name
the sign-in is listed under ("Time Trakkr CLI on <hostname>"), so that you can
tell your machines apart under Connected apps. The server keeps that name.

If the server ever refuses to renew it (you removed it, or 30 idle days
passed), tracking stops and nothing is sent or recorded until you sign in
again; the next session start opens the browser for that, once that day.

**With a pasted token, for machines with no browser:** set `TIMETRAKKR_TOKEN`
to a personal access token with the `agent` scope, from Settings, where
Cursor is started. An organization key is refused: agent time belongs
to a person. Add `TIMETRAKKR_URL` only for staging or a custom domain; it
must be `https`, except for `localhost`. When `TIMETRAKKR_TOKEN` is set it
is used in preference to a browser sign-in, and the browser never opens on its
own; `timetrakkr status` says which one is in use.

Either way, the workspace must have the agent time add-on switched on.
Otherwise the session is not tracked and nothing is sent.

The editor started from the Dock does not inherit a shell's environment, so
variables exported in a terminal profile may not reach it. That is one more
reason to prefer the browser sign-in, which needs none.

## Say what a repository's time belongs to

The same two ways as every host, and the second overrides the first: a rule an
admin sets on the platform for the repository's git remote (and optionally a
branch pattern), or a `.timetrakkr.json` at the repository root:

```json
{
  "projectId": "0b9f6a52-1c57-4f0e-9d0a-3d1f7f2f8a11",
  "taskId": "6c1d2e3f-4a5b-4c6d-8e7f-9a0b1c2d3e4f"
}
```

If the rule that applies to your branch is for a project you cannot log time
to, the session is not tracked, and `timetrakkr status` says "this branch is
mapped to a project you cannot log time to". It does not fall back to the
repository's general rule: that rule is for the other branches.

**A `.timetrakkr.json` is trusted like any other configuration committed to a
repository.** Cloning a repository that has one and starting a session in it
starts tracking, with no further question: that repository's remote and branch
are sent to your own workspace, against the project the file names. Your
workspace refuses a project you cannot log time to, so nothing goes to anyone
else's. If you do not want a repository tracked, run `timetrakkr off` in the
session, or remove the file.

**A repository with neither is not tracked: nothing about it is recorded and
nothing about it is sent, including its remote.** The repository is the first
workspace root Cursor reports.

If the mapping changes while a session is running, that day's time for the
session moves to the new project: one session has one row per day.

## What you will see

**Nothing at session start.** Cursor's `sessionStart` hook can only add text
to the agent's context; it has no way to show a line to you alone, and this
plugin does not put tracking state in front of the model. Run
`timetrakkr status` to see what a directory is tracked to.

## Commands

Two slash commands, which the agent never runs on its own:
`/timetrakkr-login` opens the browser to sign in, or again to the sign-in
already waiting, and shows the link; `/timetrakkr-status` shows
`timetrakkr status`. They are skills that have the agent run the bundled CLI.
Neither has run in a real Cursor.

The CLI itself:

| Command | What it does |
| --- | --- |
| `timetrakkr status` | What this directory is tracked to and why, and whether sign-in and the workspace switch are OK |
| `timetrakkr off` | Stop tracking a session. Time already sent stays on your timesheet for you to discard |
| `timetrakkr on` | Resume tracking a session |
| `timetrakkr estimate --hours <n> --label "<text>"` | Record how long the agent said a piece of work would take a person or a team |

Cursor documents no environment variable naming the session for the commands
the agent runs, so these act on the most recent session in the directory and
say which one they chose. With two Cursor sessions open in one repository,
that may be the wrong one.

## What is recorded

On your machine, under `~/.timetrakkr` (readable by you only): the kind and
time of each event (turn started, tool started, tool finished and how long it
ran, turn ended); for a mapped repository the directory, remote (as
`host/owner/repo`, with no credential or query string), branch, project and task; the model name; an estimate's label and
figure; the workspace's rules as last fetched; what the server last
acknowledged.

Sent to Time Trakkr, once per turn: the session id, the day, total active
seconds and turn count, first and last turn times, the project and task and
the rule they came from, the remote, the branch, the model, and an estimate if
one was recorded.

## What is never recorded

Your prompts, the agent's replies and thoughts, tool inputs and outputs, file
contents and file names, and your Cursor account email, which the hooks carry
and this program does not read. The transcript is not read at all. A token is never printed or logged, and the only file one is ever written to
is the sign-in that `timetrakkr login` stores.

## Not supported

- **Cursor cloud agents.** Cursor's documentation says `sessionStart` and
  `sessionEnd` do not run there and user-level hooks are not available; and
  your token is not on that machine. Nothing is recorded for cloud agents.
- **Token counts.** Never sent from Cursor; the product shows "not captured".
  The hooks carry none, and the transcript format is undocumented and could
  not be captured.
- **Approval waits** are not subtracted.
- **A session-start line** (see "What you will see").

## What this does not do yet

- No summary of what the session was about is sent.
- A mapping that changes mid-session moves the whole session-day to the new
  project; the day is not split between the two.
- Nothing is recorded while the workspace has agent time switched off, and
  time from before it was switched on is never sent later. "Off" includes a
  workspace whose admin has not set a default agent rate yet, so time recorded
  before the rate is set is dropped too. The session start line and
  `timetrakkr status` give the server's reason, such as "Agent time is not set
  up yet: an admin needs to set a default agent rate under Settings → Agent
  time."
