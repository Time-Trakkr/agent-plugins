---
name: timetrakkr-login
description: Sign in to Time Trakkr so this machine's agent time is recorded. Use only when the person asks to sign in to Time Trakkr or to connect it.
---

# Sign in to Time Trakkr

Run this one command and nothing else. The path is relative to the directory
this `SKILL.md` is in; if `timetrakkr` is on the `PATH`, `timetrakkr login
--detach` is the same thing.

```sh
../../bin/timetrakkr login --detach
```

It returns within a few seconds. It opens the person's browser and leaves the
sign-in waiting there in the background.

Then tell the person, in one or two short sentences, what it printed: give them
the link exactly as printed, in case the browser window did not appear, and
say that tracking starts at their next prompt once they allow it.

If it says the sign-in could not be started, or the command is refused because
it needs network access, tell them so in its words and suggest running
`timetrakkr login` in a terminal instead. Do not retry, and do not look for or
handle any credential yourself.
