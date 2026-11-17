---
name: timetrakkr-task
description: Show or choose the Time Trakkr task this session's time is recorded to. Use only when the person runs this command, never because they asked for a task in plain words.
---

# Choose the task this session is tracked to

The person ran this command to see or choose the Time Trakkr task that this
agent session's time is recorded to. Only the person chooses it: never choose
an option or a task yourself, even when one looks obvious, and never run this
command because the person asked for a task in plain words.

Run this one command, **from the workspace root**, not from this skill's
directory: it acts on the session running there. `bin/timetrakkr` is two
directories above the directory this `SKILL.md` is in, so give its absolute
path:

```sh
"<this skill's directory>/../../bin/timetrakkr" task '<word>' '<word>'
```

If `timetrakkr` is on the `PATH`, `timetrakkr task '<word>' '<word>'` in the
workspace root is the same thing.

- Pass the words the person gave with the command, each in its own single
  quotes, in their order. With no words, run it with none: it shows the
  session's current task and where it came from.
- Pass a number (`2`), `--clear`, `--whole-session` or `--from-now` that the
  person gave as it is, without quotes.
- Words are letters, digits, spaces and simple punctuation only. If the
  person's words hold quotes, backticks, dollar signs or other shell syntax,
  do not run anything: ask them to give the words again without them.

Show the person what it printed, as it is, in a code block. Then:

- **It lists tasks by number:** nothing has changed. Ask the person which
  one, or for more words, and wait for their answer. Run the command again
  with the number they give, such as `task 2`, or with their new words.
- **It lists two options, `--whole-session` and `--from-now`:** nothing has
  changed. Put both options to the person, in the command's words, and wait
  for their answer. Then run the same command again with the one option they
  chose added. If they choose neither, run nothing.
- **It fails or is refused**, including because it needs network access: say
  so in its words. Do not retry, and do not look for another way to change
  the task.

Run no other command. The task decides where the session's time is billed,
so only the person may choose it.
