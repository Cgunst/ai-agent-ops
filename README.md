# AI agent operations under change control

This project documents agent-assisted homelab operations using written work orders
and change records. Controls and results must be assessed for each run.

## The operating model

1. **Work orders:** Document scope, permitted changes and handoff for a run.
   See the [template](templates/work-order-template.md).
2. **Review:** Check each run record for the controls actually used and the results.
3. **Sensitive output:** Review artifacts and history before any possible release.

## Results from the orders

Session lengths below are from lab-docs commit timestamps; everything else is from the
run reports.

| Order | Length | Scope | Outcome |
|---|---|---|---|
| Docs + internal PKI (2026-11-17) | ~5.5 h | discovery, step-ca deployment, monitoring | CA guest live; work closed "with deferred work and limitations" — untested restore paths were recorded as gaps, not success |
| Read-only audit (2026-11-18 afternoon, 5 phases) | ~70 min | inventory + updates + vulns + repo prep | 31 assets live-enumerated; 97 upgradable packages counted with staleness caveats; 10 images + 10 guests scanned; 22 KEV CVEs; scanner written and test-run; nothing changed |


### What the agent caught


- 121,434 gitleaks matches in the docs repo: one rule hitting registry digests.
  Verified line-by-line to zero true positives before believing it.

### What the agent got wrong — and how the process caught it


- **A subagent died mid-phase** during the repo-cleanup order. The checkpoint file let
  the main session finish the phase, and the post-verification (blob counts, commit
  count) is what proved it was done correctly — not the subagent's intent.
- **An approval gate fired mid-run:** a subagent preparing the inference systemd unit
  was blocked from writing the unit file inside the guest, and the runbook recorded the
  blocked step with exact commands instead of the subagent finding a way around it.
  The main session completed the step through the allowed path before close-out — the
  gate cost a detour, not a bypass.

The pattern: agents fail confidently. The work order, the ledger, the checkpoint, the
verification-every-change rule, and a human who reads the output are what turn that
into reliable ops work.

## In this repo

- [templates/work-order-template.md](templates/work-order-template.md)
- [templates/change-ledger-template.md](templates/change-ledger-template.md)
- [templates/status-template.md](templates/status-template.md)
- [docs/guardrails.md](docs/guardrails.md) — what the agent may and may not do, and why each line exists
