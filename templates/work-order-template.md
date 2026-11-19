# Work Order <id>: <one-line goal>

**Owner:** <human>
**Executor:** <agent name> (<model/runtime, where it runs>)
**Mode:** Unattended. <Read-only / "This order makes changes — only the changes listed here.">
**Run folder:** `<path>/runs/<id>/` (raw output in a location **outside** every repo)

Execute this work order from top to bottom. <One paragraph of context: what came before,
what this order is for.>

---

## 0. Ground rules

1. **Only make the changes this order lists.** Everything else stays read-only.
2. **Never do any of these:** <list the absolute bans: reboots, service restarts on
   specific critical hosts, pushes/publishing, credential changes, deleting
   snapshots…>.
3. **Snapshot before every change** (name pattern `<phase><date>`). Never delete or
   roll back a snapshot — that's the human's call.
4. **Stop on failure within a phase.** Record host, command, error, and resulting
   state under "Needs Cody". Do not roll back. Continue with phases that don't depend
   on the failed one.
5. **Secrets:** never print, log, or commit a secret. Mask anything found
   (first 4 characters, then `…`).
6. **Don't stop to ask <human> anything.** Log blockers under "Needs Cody" with the
   exact command they would run, and keep going.
7. **Keep context lean:** raw output to files; read back only what's needed. Delegate
   phases to subagents when each phase is self-contained.
8. **Checkpoint:** keep `STATUS.md` updated after every phase
   (`done` / `partial` / `blocked` + output files). On session restart: read
   `STATUS.md`, resume from the first phase that isn't `done`.
9. **Commit** at the end of each phase: `<order id>: phase N <name>`.
10. **Identity hazards:** <name-your-collision-rules — e.g. "IDs repeat across
    hosts; always state the host with the ID">.

---

## Phase 1: <name>

<Goal sentence.>

**Targets / inputs:** <list>

**Steps:**
1. <record before-state>
2. <snapshot>
3. <the change, exact command>
4. <verify: the specific, functional checks — not just "it started">

**Output:** `<run folder>/reports/<file>.md` — <what the artifact must contain>

## Phase N: Close out

1. Finish `STATUS.md`.
2. Write `<run folder>/REPORT.md`: one paragraph per phase with key numbers; **every
   change made** (host, command, before → after, snapshot); every snapshot created
   with its cleanup command; the human's next steps in order; the full "Needs Cody"
   list with exact commands.
3. Final commit to each repo.
4. <notification requirement, or note why it couldn't be sent>
5. End the session.

---

## Needs Cody (maintained during the run)

| # | Blocker / decision | Exact command or decision text |
|---|---|---|
| 1 | | |
