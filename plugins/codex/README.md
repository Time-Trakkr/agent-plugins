# Time Trakkr for Codex

Records how long Codex worked in a repository and reports it to Time Trakkr,
where it waits on your timesheet for you to review.
At the end of the day it becomes a time entry on its own if your workspace
accepts automatically, which it does unless an admin has turned that
off. Time mapped by a `.timetrakkr.json` file in the repository, and time a
cloud agent checks in, is never accepted automatically.

This covers the Codex CLI, interactive and `codex exec`. See "Not supported"
below for where it does not work.

## What is measured

A turn starts when you submit a prompt and ends when the agent stops. Agent
time is the sum of turns. The time between turns, while you read or are away,
is not counted. A turn you interrupt is closed at the moment you interrupt it:
Codex reports interrupts.

Time the agent spends waiting for you to approve a command **is** counted on
Codex. Codex documents a hook for approval prompts, but it could not be
confirmed against a real session, so it is not used yet.

## Install

Published from a public marketplace repository, with the CLI bundled, so
there is nothing to build:

```sh
codex plugin marketplace add Time-Trakkr/agent-plugins && codex plugin add timetrakkr@timetrakkr
```

Then start Codex and review and trust the plugin's hooks with `/hooks`:
Codex does not run a plugin's hooks until you have. It needs Node 20 or later
on your `PATH`.

Installing from a local copy of that layout was tested with Codex CLI 0.160.0
in a throwaway `CODEX_HOME`: Codex picked the Codex catalog and installed the
plugin with `bin/timetrakkr` intact. A Codex session on that install was not
run.

## Install for development

The plugin runs the CLI as one file, `bin/timetrakkr`. That file is a build
artifact and is not checked in, so a fresh checkout has no `bin/` and the
plugin's hooks do nothing until it is built. From the repository root, once
and again after any change to `packages/agent-cli`:

```sh
pnpm --filter @timetrakkr/cli build:plugin
```

Then build the publishable layout and add it as a local marketplace
(`node publishing/agent-plugins/build-layout.mjs <dir> --version 0.0.1`, then
`codex plugin marketplace add <dir>` and `codex plugin add timetrakkr@timetrakkr`).
Codex copies a plugin into its cache, so after a change, build and add it
again.

To use `timetrakkr status`, `off`, `on` and `estimate` from a terminal, and
for the estimate skill to find it, `timetrakkr` must also be on your
`PATH`, for example a link to the installed plugin's `bin/timetrakkr` in
`~/.local/bin`.

## Sign in

**Usually there is nothing to do.** The first session on a machine that is not
signed in, in a repository that could be tracked, opens your browser to sign
in to Time Trakkr. Sign in, pick your workspace and press Allow; tracking
starts at your next prompt. There is no address to type: the sign-in goes
through `https://timetrakkr.com`, which lists every workspace you belong to.

The browser opens on its own at most once a day per machine, never while a
sign-in is already waiting, and never after `timetrakkr logout` until you sign
in again. If you closed it, ask Codex to use the `timetrakkr-login` skill, which
runs the bundled CLI to open the browser again and prints the link, or run
`timetrakkr login` in a terminal. The skill needs network access: if
the sandbox refuses it, use the terminal. `timetrakkr-status` says what the
session is tracked to.

From a terminal it is `timetrakkr login`. For a workspace on staging or on a
custom domain, sign in once with `timetrakkr login --url <address>`; the
address is remembered.

The sign-in is kept in an owner-only file under `~/.timetrakkr`, renews
itself, and lasts as long as it is used at least once every 30 days. It can
reach the two agent-time routes and nothing else in your workspace.
`timetrakkr logout` revokes it and deletes it from this machine; you can also
remove it under Integrations → Connected apps.

Signing in sends this computer's hostname to the server, as part of the name
the sign-in is listed under ("Time Trakkr CLI on <hostname>"), so that you can
tell your machines apart under Connected apps. The server keeps that name.

If the server ever refuses to renew it (you removed it, or 30 idle days
passed), tracking stops and nothing is sent or recorded until you sign in
again; the next session start opens the browser for that, once that day.

**With a pasted token, for machines with no browser:** set `TIMETRAKKR_TOKEN`,
where Codex is started, to a personal access token with **Agent time only**
ticked, from Integrations → Personal access tokens. An organization key is refused: agent time belongs
to a person. Add `TIMETRAKKR_URL` only for staging or a custom domain; it
must be `https`, except for `localhost`. When `TIMETRAKKR_TOKEN` is set it
is used in preference to a browser sign-in, and the browser never opens on its
own; `timetrakkr status` says which one is in use.

Either way, the workspace must have the agent time add-on switched on.
Otherwise the session is not tracked and nothing is sent.

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
nothing about it is sent, including its remote.** The request that fetches the
workspace's rules carries your token and nothing about the repository.

If the mapping changes while a session is running, the change is picked up at
your next prompt, and that day's time for the session moves to the new
project: one session has one row per day. If the repository stops being
mapped, what was already recorded is posted to the old project and recording
stops.

## What you will see

One line when a session starts, sent to Codex as a `systemMessage`, which its
documentation says is shown to you as a notice:

```
Time Trakkr: tracking to Acme / Website rebuild / Development. Run "timetrakkr off" to stop.
Time Trakkr: not tracked (this repository is not mapped to a project).
```

It is not added to the agent's context; a real session confirmed that.
Whether the interactive UI shows it was not seen, and `codex exec` does not
print it. `timetrakkr status` always tells you.

## Commands

Two skills, for when you ask: `timetrakkr-login` opens the browser to sign in,
or again to the sign-in already waiting, and shows the link;
`timetrakkr-status` shows `timetrakkr status`. Codex runs the bundled CLI for
them as an ordinary command, so its sandbox and approval rules apply. Neither
was run in a real Codex session.

The CLI itself:

| Command | What it does |
| --- | --- |
| `timetrakkr status` | What this directory is tracked to and why, and whether sign-in and the workspace switch are OK |
| `timetrakkr off` | Stop tracking this session. Time already sent stays on your timesheet for you to discard |
| `timetrakkr on` | Resume tracking this session |
| `timetrakkr estimate --hours <n> --label "<text>"` | Record how long the agent said a piece of work would take a person or a team |

Run by the agent, these act on the session Codex names in `CODEX_SESSION_ID`.
Run from a plain terminal, they pick the most recent session in the directory
and say which one they chose.

## Estimated effort

When the agent tells you how long something would take a person or a team, the
`record-estimate` skill has it run `timetrakkr estimate` with that figure and
a short name for the work. It is sent once, with the session's next post, and
shown beside the agent's own time for comparison. It is never billed. For a
team the agent records total effort, people times duration.

In testing the agent used the skill when asked a planning question, and did
not when told not to read any files: Codex reads a skill's file before
following it.

## What is recorded

On your machine, under `~/.timetrakkr` (readable by you only):

- for each session, the kind and time of each event: turn started, tool
  started, tool finished, turn ended or interrupted
- for a mapped repository: the working directory, repository root, remote
  (as `host/owner/repo`, with no credential, query string or fragment), branch, and the project and task
- token counts per model and per day, read from Codex's rollout file when a
  turn or the session ends, with the ids of recent responses so none is
  counted twice
- the label and figure of an estimate
- the workspace's repository rules, as last fetched
- what the server last acknowledged, so nothing is counted twice

Sent to Time Trakkr, once per turn: the session id, the day, total active
seconds and turn count for that day, the first and last turn times, the
project and task and the rule they came from, the remote, the branch, the
model, token counts per model, and an estimate if one was recorded.

## What is never recorded

Your prompts, the agent's replies, tool inputs and outputs, file contents and
file names. The rollout file is read for token counts, the model name and
response ids only; none of its text is kept or sent, and its location is not
stored. The one piece of free text kept is the label given to
`timetrakkr estimate`. A token is never printed or logged, and the only file one is ever written to
is the sign-in that `timetrakkr login` stores.

## Not supported

- **Codex cloud.** A cloud task runs on OpenAI's machines, where this plugin's
  hook scripts are not installed and your token is not present, so the plugin
  records nothing there. A cloud task can check in its time instead: see
  [agent time for agents in the cloud](https://timetrakkr.com/help/integrations/agent-time-cloud).
- **The Codex IDE extension.** Codex's documentation says plugins are not
  available there.
- **Approval waits** are not subtracted (see "What is measured").
- **Sandboxed commands writing state.** `timetrakkr estimate`, `off` and `on`
  run by the agent write under `~/.timetrakkr`. In testing the state directory
  was under the temporary directory, which Codex's `workspace-write` sandbox
  allows. Whether the sandbox lets the agent write to `~/.timetrakkr` was not
  tested; if it does not, the command fails, the agent carries on, and the
  estimate is not recorded. Hooks are not affected: in testing they reached
  the server while the agent's own commands had network access disabled.

## What this does not do yet

- No summary of what the session was about is sent.
- A mapping that changes mid-session moves the whole session-day to the new
  project; the day is not split between the two.
- Token counts are "not captured" rather than zero whenever the rollout file
  cannot be read or has a usage record the CLI does not understand: Codex says
  that file is not a stable interface. The same applies for the rest of a
  session in which recorded time had to be dropped.
- Subagents were not tested. Codex's documentation says they share the
  parent's session id, so their tool activity should land inside the parent's
  turn; that is unconfirmed.
- Nothing is recorded while the workspace has agent time switched off, and
  time from before it was switched on is never sent later. "Off" includes a
  workspace whose admin has not set a default agent rate yet, so time recorded
  before the rate is set is dropped too. The session start line and
  `timetrakkr status` give the server's reason, such as "Agent time is not set
  up yet: an admin needs to set a default agent rate under Settings → Agent
  time." Time recorded
  while the server was merely unreachable is kept and sent when it is back.

## If something is wrong

Hooks never print and never fail, so a problem shows up as missing time.
`timetrakkr status` reports the last error for the session, and
`~/.timetrakkr/hook.log` has anything a hook could not do.
