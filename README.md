# AI agent operations under change control

How I run a self-hosted AI agent as an actual ops teammate instead of a toy: local
models, scoped credentials, written work orders, snapshots before every change, and a
human who reviews the output before it counts. Everything here comes out of real runs
in my homelab (sanitized: role names, example addressing); the numbers are from the run
logs and git history.

## The setup

- **Nothing leaves the machine.** The agent runs in an unprivileged LXC container and
  is wired to a **local Mixture-of-Experts inference server** on the same LAN — a GPU
  endpoint loading MoE weights with experts in system RAM (tens of GB; the unit file,
  health checks, and restart policy around it are documented in the `homelab` repo).
  No cloud API keys in the loop.
- **The agent's hands are scoped:** read-only hypervisor API tokens (auditor role), a
  Checkmk automation user with a least-privilege role, SNMPv3 read-only on the switch,
  SSH as a named key, and per-repo deploy keys for git. No global admin anywhere.
- **The agent talks to me through chat** (a Discord channel) for reports and alerts —
  and the first lesson was that alert *delivery* is itself a system that needs
  monitoring: jobs delivered to a never-configured platform failed silently for days.

## The operating model

1. **Work orders, not vibes.** Every non-trivial session executes a written order:
   owner, executor, mode (read-only or which changes are allowed), ground rules,
   numbered phases with named output files, and a hard "Needs Cody" section for
   anything the agent may not do. ([template](templates/work-order-template.md))
2. **Read-only first.** Discovery and reporting orders never write. Change orders
   change *only what the order lists* — everything else stays read-only by default.
3. **Snapshot before every change; stop on failure; never auto-rollback.** A failed
   change stops the phase, records guest/command/error/service state, and leaves the
   snapshot for the human. The agent that made the mess is never the one that decides
   "rollback and try again" — rollback is a human call with a maintenance window.
4. **Change ledger + checkpoint file.** Every mutation lands in a ledger table (host,
   command, before→after, snapshot name, verification). A `STATUS.md` checkpoint lets a
   dead or restarted session resume from the first unfinished phase instead of
   re-running done ones.
5. **Human-only actions.** Node reboots, risky upgrades (release upgrades, anything
   that removes packages), publishing to the internet, credential rotation, deleting
   snapshots. The agent *prepares* these — runbooks, pre-checked scripts, recorded
   rollback digests — and stops.
6. **Raw output stays out of git.** Scan dumps, API responses, and probe output go to
   a run-data directory outside every repo; commits carry the report, not the corpus.

## Results from the first three orders

Session lengths below are from lab-docs commit timestamps; everything else is from the
run reports.

| Order | Length | Scope | Outcome |
|---|---|---|---|
| Docs + internal PKI (2026-11-17) | ~5.5 h | discovery, step-ca deployment, monitoring | CA guest live; work closed "with deferred work and limitations" — untested restore paths were recorded as gaps, not success |
| Read-only audit (2026-11-18 afternoon, 5 phases) | ~70 min | inventory + updates + vulns + repo prep | 31 assets live-enumerated; 97 upgradable packages counted with staleness caveats; 10 images + 10 guests scanned; 22 KEV CVEs; scanner written and test-run; nothing changed |
| Change order (2026-11-18 evening, 8 phases) | ~2 h 05 min | patching + prep + corrections + cleanup | 7 guests patched (245 packages, 6 held back, all services verified after), zero rollbacks needed; vuln report corrected; history purged (115 commits intact, 0 raw-data blobs left) |

### What the agent caught

- A monitoring image upgrade crash-looping 10 minutes after start — root-caused to a
  data volume shadowing the new image's install tree, rolled back to the recorded
  digest, and the whole story written up with fix options. (The upgrade itself is still
  an open item — the rollback is what kept monitoring up.)
- A stale *docs-vs-reality* class: boot flags documented as set but actually default-off,
  a RAM value with a typo, a config file on disk that the running app never reads.
- A KEV finding whose premise was stale: the vulnerable daemon ("telnetd") wasn't even
  installed — only the client.
- 121,434 gitleaks matches in the docs repo: one rule hitting registry digests.
  Verified line-by-line to zero true positives before believing it.

### What the agent got wrong — and how the process caught it

- **Ranked a KEV OpenSSH CVE #1 fleet-wide.** It was a musl/no-PAM-only bug; zero
  guests applied. Cody asked "is this actually applicable?" and one probe per guest
  closed it. The correction became code: the scanner now probes and filters (see the
  [vuln-management-lab repo](https://github.com/Cgunst/vuln-management-lab)).
- **A confident false escalation** ("unpatched for ~2 years") born from stale package
  databases on the scanning side. Pre/post captures disagreed with the claim; the
  re-verification won.
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
