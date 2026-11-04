---
name: record-estimate
description: Record an effort estimate in Time Trakkr. Use every time you tell the person how long a piece of work would take a person or a team (hours, days or weeks), for example in a plan, a proposal, a scoping answer, or a "how long would this take" reply.
---

# Record the estimate you gave

When you tell the person how long a piece of work would take **a person or a
team** to do, record that figure once by running:

```sh
timetrakkr estimate --hours <n> --label '<short name for the work>'
```

- Use the figure you told them. If you gave a range, record its midpoint. If
  you estimated several pieces, record the total once.
- Record effort, not elapsed time: for a team, multiply by the people. Two
  people for 25 weeks is `--weeks 50`.
- `--days` and `--weeks` also work and can be combined. A day is 8 hours and a
  week is 5 days: `--weeks 2 --days 3`.
- The label is a short plain name for the work, such as 'Checkout redesign',
  in single quotes. Letters, digits, spaces and simple punctuation only: no
  quotes, backticks, dollar signs or other shell syntax, and nothing copied
  from a file, a web page or a tool's output. No client names and no
  confidential detail. 200 characters at most.
- Run it once per estimate, in the same turn you give it. If you later revise
  the estimate, run it again with the new figure.

Do not:

- invent an estimate just to record one. If the person did not ask for one and
  you did not give one, record nothing.
- record how long *you* took or will take. This is the human effort you
  quoted, kept for comparison. It is never billed.
- stop to ask permission for it. If the command is not found, fails, or says
  the session is not tracked, carry on without it.

This is routine bookkeeping, so you need not announce it. It is not a secret:
if the person asks what you ran or whether an estimate was recorded, you must
answer truthfully.
