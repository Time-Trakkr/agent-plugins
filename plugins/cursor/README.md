# Time Trakkr for Cursor

Records how long Cursor's agent worked in a repository and reports it to Time
Trakkr, where it waits on your timesheet for you to review. Nothing becomes a
time entry on its own.

## Status

Supported in the Cursor editor's agent chat and in the Cursor CLI,
`cursor-agent`, interactive and with `-p`. Both were checked end to end on
macOS on 2026-11-06: the editor with Cursor 3.29.9 (plugin installed under
`~/.cursor/plugins/local`, editor restarted), the CLI with `cursor-agent`
2026.11.05. Windows has not been checked. If a session records nothing, check
`~/.timetrakkr/hook.log` and run `timetrakkr status` in the repository.

The command-line tool behind this plugin is the same one the Claude Code and
Codex plugins use.

## What is measured

Agent time is the sum of turns, one per prompt. The time between turns is not
counted.

**In the editor, whole turns are counted.** The editor runs a plugin's
`beforeSubmitPrompt` and `stop` hooks, so a turn runs from the prompt to the
agent stopping. It also reports the real model rather than "default". Quitting
the editor does not run `sessionEnd`; nothing depends on it, because each turn
is sent when it ends.

**In the Cursor CLI only part of each turn is counted** until `timetrakkr
cursor setup` is run. The CLI runs a
plugin's `preToolUse`, `postToolUse`, `postToolUseFailure`, `sessionStart` and
`sessionEnd` hooks, but not its `beforeSubmitPrompt` or `stop` hooks: it runs
those only when `~/.cursor/hooks.json` or the project's `.cursor/hooks.json`
defines the same event. Every hook carries `generation_id`, new for each
prompt, so a tool call from a generation not seen before opens its turn, and
the turn closes at its last tool call. The agent's thinking before its first
tool call and its answer after its last are left out, and a turn with no tool
call counts nothing. In the sessions it was checked against, that was a
little under half of the prompt-to-stop time.

**`timetrakkr cursor setup` fixes that.** Run it once (`npx @timetrakkr/cli
cursor setup` if `timetrakkr` is not on your `PATH`). It shows the two entries
it will add to `~/.cursor/hooks.json` and asks first:

```json
"beforeSubmitPrompt": [{ "command": "node -e 0 timetrakkr-cursor-full-turns" }]
"stop": [{ "command": "node -e 0 timetrakkr-cursor-full-turns" }]
```

They do nothing themselves; with them in place the CLI runs this plugin's own
`beforeSubmitPrompt` and `stop` hooks. `node -e 0` rather than `true`, because
on Windows Cursor runs hooks through PowerShell, which has no `true`; Windows
has not been checked yet. Your other hooks are kept, the file as
it was is saved as `hooks.json.timetrakkr-backup`, and `--undo` removes exactly
those entries. A `hooks.json` it cannot parse is left alone. `timetrakkr
status` says whether it is set up. Checked on staging: the same kind of
two-prompt session recorded 8 seconds without it and 19 with it.

Where `beforeSubmitPrompt` and `stop` do run, a turn starts at the prompt and
ends when the agent stops, whether it completed, was aborted or hit an error.

Time the agent spends waiting for you to approve something **is** counted on
Cursor: its hooks have no signal for it.

## Install

Published from a public marketplace repository, with the CLI bundled, so
there is nothing to build. Once, in a terminal:

```sh
cursor-agent plugin marketplace add https://github.com/Time-Trakkr/agent-plugins
```

then start `cursor-agent`, type `/plugins`, open **Marketplace**, pick
`timetrakkr` and choose **Install for you**. It is installed under
`~/.cursor/plugins/cache/timetrakkr`. This works on a personal account; an
admin can also import the repository as a team marketplace (Teams or
Enterprise), which has not been tried.

**Upgrading.** `cursor-agent plugin marketplace update` does not move an added
marketplace to a newer release (checked with `cursor-agent` 2026.11.05: it kept
1.0.2 after 1.1.0 was published). Remove the marketplace with
`cursor-agent plugin marketplace remove timetrakkr`, add it again, then
uninstall and reinstall `timetrakkr` from `/plugins`. 1.0.2 records nothing in
the Cursor CLI; 1.1.0 is the first release that does.

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
`/timetrakkr-status`, or `timetrakkr status` in a terminal, to see what a
directory is tracked to.

The CLI does not always run a plugin's `sessionStart`, so the browser may not
open by itself to sign you in. `/timetrakkr-login` opens it.

## Commands

Two slash commands, which the agent never runs on its own:
`/timetrakkr-login` opens the browser to sign in, or again to the sign-in
already waiting, and shows the link; `/timetrakkr-status` shows
`timetrakkr status`. They are skills that have the agent run the bundled CLI.
`/timetrakkr-status` has run in the Cursor CLI; `/timetrakkr-login` has not.

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
and this program does not read. The transcript is not read at all.

Cursor also runs installed Claude Code plugins' hooks, with Cursor's own
payloads. If you have the Claude Code plugin installed too, it recognizes a
Cursor payload and records nothing, so a Cursor session is never counted
twice. A token is never printed or logged, and the only file one is ever written to
is the sign-in that `timetrakkr login` stores.

## Not supported

- **Cursor cloud agents.** Cursor's documentation says `sessionStart` and
  `sessionEnd` do not run there and user-level hooks are not available; and
  your token is not on that machine. Nothing is recorded for cloud agents.
- **Token counts.** Never sent from Cursor; the product shows "not captured".
  `stop` carries token counts, but the CLI does not run a plugin's `stop`, and
  under Auto the hooks name no model to price them at.
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
