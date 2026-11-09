# Time Trakkr plugins for coding agents

Records how long your coding agent worked in a repository your workspace has
connected, and reports it to [Time Trakkr](https://timetrakkr.com), where it
waits on your timesheet for you to review. At the end of the day it becomes a
time entry on its own if your workspace accepts automatically, which it does
unless an admin has turned that off. Version 1.2.1.

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

**Cursor (the editor and the `cursor-agent` CLI)**

Install it with the Cursor CLI. Run this once in a terminal:

```sh
cursor-agent plugin marketplace add https://github.com/Time-Trakkr/agent-plugins
```

then start `cursor-agent`, type `/plugins`, open **Marketplace**, pick
`timetrakkr` and choose **Install for you**. Restart the Cursor editor
afterwards, so it loads the plugin. Then, once, so that the Cursor CLI counts
whole turns:

```sh
npx @timetrakkr/cli cursor setup
```

It shows the two entries it will add to `~/.cursor/hooks.json` and asks first.
`npx @timetrakkr/cli cursor setup --undo` takes them out again.

To upgrade, `cursor-agent plugin marketplace update` does not move to a newer
release: run `cursor-agent plugin marketplace remove timetrakkr`, add the
marketplace again with the command above, then uninstall and reinstall
`timetrakkr` from `/plugins`.

**Agents in the cloud**

A cloud agent does not run these plugins. Claude Code cloud sessions can
measure their time with `npx @timetrakkr/cli init --cloud`, and any cloud agent
can check in instead. See
[agent time for agents in the cloud](https://timetrakkr.com/help/integrations/agent-time-cloud).

## Sign in

Start a session in a repository your workspace has connected. If this machine
is not signed in, your browser opens once to sign in: sign in to Time Trakkr,
pick your workspace, and press Allow. Tracking starts at your next prompt.

If you closed that window, sign in from inside the session:

- Claude Code: `/timetrakkr:login`
- Codex: ask it to use the `timetrakkr-login` skill, or run
  `~/.codex/plugins/cache/timetrakkr/timetrakkr/1.2.1/bin/timetrakkr login`
- Cursor: `/timetrakkr-login`

Your workspace on a custom domain or on staging: sign in with
`timetrakkr login --url <address>` once; the address is remembered.

## What is recorded and what is not

Each plugin's README says exactly what is recorded, what is sent and what
never is. In short: turn start and end times, the repository's remote and
branch for a connected repository, and token counts. Never your prompts, the
agent's replies, tool input or output, or file contents.

## License

MIT. See `LICENSE`; each plugin carries a copy.
