---
name: timetrakkr-status
description: Show what Time Trakkr is tracking this session to, and whether its sign-in works. Use only when the person asks about Time Trakkr's status.
---

# Time Trakkr status

Run this one command and nothing else, **from the workspace root**, not from
this skill's directory: it reports on the directory it runs in. `bin/timetrakkr`
is two directories above the directory this `SKILL.md` is in, so give its
absolute path:

```sh
"<this skill's directory>/../../bin/timetrakkr" status
```

If `timetrakkr` is on the `PATH`, `timetrakkr status` in the workspace root is
the same thing.

Show the person what it printed, as it is, in a code block. If a line says
something is wrong, add one sentence saying what to do about it. If the
command is refused because it needs network access, say so.
