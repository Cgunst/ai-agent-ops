# STATUS — <order id>

Executor: <agent> on <host>
Started: <date time TZ> · Last update: <date time TZ>

## Phase status

- [x] Phase 1: <name> — DONE <time>
  - <one line of key numbers / findings>
  - outputs: `<files>` · commits: `<hashes>`
- [x] Phase 2: <name> — PARTIAL <time>
  - <what was done, what wasn't, and why>
- [ ] Phase 3: <name> — not started
  - **resume here**: first thing to run is `<exact command>`

## Resume notes

If the session restarts: this file is the truth. Resume from the first phase that is
not `done`. Re-verify preconditions of the resume phase (host reachable, snapshot
absent/present) before acting.

## Needs Cody

| # | Blocker / decision needed | Exact command or choice |
|---|---|---|
| 1 | | |

## Blockers / log

- <time>: <anything weird, dead subagents, retried steps — things a resuming session
  must know>
