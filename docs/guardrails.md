# Guardrails: what the agent may and may not do

Every rule here exists because of a specific failure mode we hit or designed around.
"May" and "may not" are written into the ground-rules section of every work order; this
is the reasoning behind them.

## The agent may

- **Read anything in scope, through read-only interfaces.** API GETs, `pct config`,
  `ss`, package queries, agent output. Discovery is free; that's where most of the
  agent's value was (a 70-minute read-only order produced the first complete,
  discrepancy-flagged inventory of 31 assets).
- **Write where the order says, and only there.** One phase, one blast radius.
- **Snapshot before its own changes**, every time, named `<phase><date>`.
- **Prepare human-only work fully:** runbooks with exact commands, check scripts
  pre-tested with their expected FAILs recorded, rollback digests recorded before an
  upgrade starts. Preparation is mechanical and thorough; that's what the agent is
  better at than me.
- **Stop and record.** "Needs Cody" with the exact command is a successful outcome,
  not a failure. A logged blocker beats an unauthorized workaround every time.
- **Finish another subagent's phase** if that subagent dies — from the checkpoint file
  and verification, not from trusting the dead agent's intentions.

## The agent may not

- **Reboot or power-cycle nodes, or restart the services it runs on.** Two reasons:
  the agent's own runtime sits on the infrastructure it operates (self-kill loop), and
  a reboot is a multi-service outage event — a scheduling decision, not a technical
  one. It prepares the reboot runbook and the post-reboot check instead.
- **Roll back automatically.** A failed change stops the phase and freezes the state
  for diagnosis. Auto-rollback destroys evidence and can compound: the classic case is
  the monitoring upgrade that crash-looped because of the *volume layout* — rolling
  forward or undoing "just the tag" repeatedly would have churned a production-ish
  monitoring server until a human read the entrypoint logs.
- **Do release upgrades or removal-class changes at all.** `upgrade` yes,
  `full-upgrade`/`dist-upgrade` never in an automated pass; anything whose simulation
  says "will remove N packages" goes to the human with the simulation attached.
- **Touch credentials, firewall rules, or keys.** Rotate-when-compromised is a human
  incident process; while-read-only-access-doesn't-exist-yet is a documented debt, not
  an invitation to improvise.
- **Push to remotes or publish anything.** The GitHub work order's own rule: nothing
  becomes public by the agent's action. Repos are created private by the human; a human
  flips visibility after reading every line.
- **Delete snapshots, history, or backups.** The purge/rewrite operations the agent
  *does* run (git history scrubbing, moving raw data out of repos) require a mirror
  backup taken first, a before/after count assertion (115 commits intact, 0 blobs
  left), and they never touch the snapshot layer.
- **Print a secret, ever.** Mask format: first 4 chars then `…`. Keys are read
  programmatically into place (env files mode 600) without ever appearing in output,
  commits, or this documentation.
- **Decide applicability on its own authority for anything destructive.** Scanner
  output is an input to a judgment, not the judgment: the KEV CVE ranked #1 by
  automation was applicable to zero hosts. High-severity claims that would justify
  risky action get independently verified first — that rule exists in writing now.

## Why this shape

The failure mode of capable agents isn't laziness, it's **confident action on wrong
premises** — ranking an inapplicable CVE #1, escalating "2 years unpatched" from a
stale database, running `pct` against the wrong node because two nodes share VMIDs.
So the control set isn't about limiting intelligence; it's about:

1. **narrowing blast radius** (read-only defaults, per-phase scope, scoped creds),
2. **keeping a human decision point in front of every irreversible action**
   (reboots, publishing, deletions), and
3. **making every claim verifiable after the fact** (ledger, checkpoint, raw-outside-
   git, verify-every-change).

Three orders in, that system has produced real ops work with zero unplanned outages
and zero rollbacks-by-surprise — and it caught its own agent being wrong repeatedly,
which is the only metric that matters.
