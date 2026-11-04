---
name: login
description: Sign in to Time Trakkr so this machine's agent time is recorded. Only when the person asks to sign in.
allowed-tools: Bash(*/bin/timetrakkr login --detach*)
disable-model-invocation: true
---

The person asked to sign in to Time Trakkr. This already ran:

!`"${CLAUDE_PLUGIN_ROOT}/bin/timetrakkr" login --detach 2>&1`

Tell the person, in one or two short sentences, what the output above says.
If it opened a browser, give them the link exactly as printed, in case the
window did not appear, and say that tracking starts at their next prompt once
they allow it. If it says the sign-in could not be started, say why in its
words. Do not run anything else.
