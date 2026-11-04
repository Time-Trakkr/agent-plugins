---
name: status
description: Show what Time Trakkr is tracking this session to, and whether sign-in works. Only when the person asks.
allowed-tools: Bash(*/bin/timetrakkr status*)
disable-model-invocation: true
---

The person asked for Time Trakkr's status. This already ran:

!`"${CLAUDE_PLUGIN_ROOT}/bin/timetrakkr" status 2>&1`

Show the person the output above as it is, in a code block. If a line says
something is wrong, add one sentence saying what to do about it. Do not run
anything else.
