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

2. **Secrets:** never print, log, or commit a secret. Mask anything found
   (first 4 characters, then `…`).
3. **Don't stop to ask <human> anything.** Log blockers under "Needs Cody" with the
   exact command they would run, and keep going.
4. **Keep context lean:** raw output to files; read back only what's needed. Delegate
   phases to subagents when each phase is self-contained.
5. **Checkpoint:** keep `STATUS.md` updated after every phase
   (`done` / `partial` / `blocked` + output files). On session restart: read
   `STATUS.md`, resume from the first phase that isn't `done`.
6. **Commit** at the end of each phase: `<order id>: phase N <name>`.
7. **Identity hazards:** <name-your-collision-rules — e.g. "IDs repeat across
    hosts; always state the host with the ID">.

---

## Phase N: Close out


1. Final commit to each repo.
2. <notification requirement, or note why it couldn't be sent>
3. End the session.

---

## Needs Cody (maintained during the run)

| # | Blocker / decision | Exact command or decision text |
|---|---|---|
| 1 | | |
