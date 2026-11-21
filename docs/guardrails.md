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

- **Stop and record.** "Needs Cody" with the exact command is a successful outcome,
  not a failure. A logged blocker beats an unauthorized workaround every time.
- **Finish another subagent's phase** if that subagent dies — from the checkpoint file
  and verification, not from trusting the dead agent's intentions.

## The agent may not


- **Do release upgrades or removal-class changes at all.** `upgrade` yes,
  `full-upgrade`/`dist-upgrade` never in an automated pass; anything whose simulation
  says "will remove N packages" goes to the human with the simulation attached.
- **Touch credentials, firewall rules, or keys.** Rotate-when-compromised is a human
  incident process; while-read-only-access-doesn't-exist-yet is a documented debt, not
  an invitation to improvise.
- **Push to remotes or publish anything.** The GitHub work order's own rule: nothing
  becomes public by the agent's action. Repos are created private by the human; a human
  flips visibility after reading every line.

- **Print a secret, ever.** Mask format: first 4 chars then `…`. Keys are read
  programmatically into place (env files mode 600) without ever appearing in output,
  commits, or this documentation.




