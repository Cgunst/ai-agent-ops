# Change ledger — <order id>

Every mutation made by this run, in order. Verification means the specific check from
the work order's step 4 was executed **after** the change and its result recorded —
not "the command exited 0".

| # | Time (TZ) | Host / guest | Change (command, shortened) | Before → After | Snapshot | Verified how / result |
|---|-----------|--------------|-----------------------------|----------------|----------|------------------------|
| 1 | | | | | | |

Rules this ledger enforces:

- **No snapshot → no change.** If the change isn't snapshot-able (config file, crontab),
  a timestamped copy-backup file plays the snapshot role and is named in the column.
- **Before → After is literal** (`6166 MiB typo → 6144 MiB`, `p19 digest 832b… → attempted
  p20 → rolled back to p19 digest, healthy`). "Updated" is not a before/after.
- **Rollbacks get their own row**, pointing at the failure row they undo, and the state
  after rollback is verified and recorded (hosts monitored, services up, config intact).
- **Snapshot cleanup is never in this ledger's scope** — the ledger records the cleanup
  *commands* for the human to run after the system is proven.
- Failed phases that changed nothing get a row saying `no change` with the error, so
  the ledger doubles as the "what did NOT happen" record.

## Snapshots created (human deletes, never the agent)

| Snapshot | Created on | Cleanup command |
|---|---|---|
| | | |
