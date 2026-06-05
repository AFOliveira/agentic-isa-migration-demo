# Job spec seeds (E2E chain)

## Task

`<task-id>` — set to the active MULTIAGENT task.


Bootstrap:

```bash
multiagent agent job create isa-migration-plan -r planner -t <task-id> .multiagent/specs/isa-migration-plan.md
# planner creates:
multiagent agent job create isa-udb-audit -r udb-auditor -t <task-id> .multiagent/specs/isa-udb-audit.md
```

Hub-and-spoke: the **planner** authors and dispatches each job from the matching
`isa-*.md` template (it owns the WHAT); agents do their role's HOW and notify the
planner when done — they do **not** create each other's jobs. The one exception is
**rtl → dv** (a sub-chain: rtl creates isa-dv directly, in rtl's worktree). See
`roles/planner.md` → Dispatch model.

| Job ID | Role | Seed |
|--------|------|------|
| isa-udb-audit | udb-auditor | isa-udb-audit.md |
| isa-encoding-analyst | encoding-analyst | isa-encoding-analyst.md |
| isa-compliance | compliance | isa-compliance.md |
| isa-toolchain | toolchain | isa-toolchain.md |
| isa-rtl | rtl | isa-rtl.md |
| isa-dv | dv | isa-dv.md (directed core-et DV, independent of rtl) |
| isa-simulator | simulator | isa-simulator.md |
| isa-zephyr | zephyr | isa-zephyr.md |
| isa-integrate | integrator | isa-integrate.md |
| isa-run-sysemu | sysemu-run | isa-run-sysemu.md |
| isa-run-verilator | verilator-run | isa-run-verilator.md |
| isa-review | reviewer | isa-review.md (pre-release gate, after Phase 3; reject → planner routes fix) |
| isa-release-integrate | committer | isa-release-integrate.md (commit/push touched repos + release manifest) |
| isa-release-review | reviewer | isa-release-review.md (final pushed-state gate) |
| isa-document | documenter | isa-document.md |

Shared workspace: `harness/jobs/isa-toolchain/workspace/aiflow/env.sh` (from repo root; or `jobs/isa-toolchain/workspace/aiflow/env.sh` after `cd harness`)


## MULTIAGENT notes

- Job specs are passed as a file path to `multiagent agent job create`.
- Workers must create a `role=planner` notification job on the same task before `job done` / `job fail` / `job release` (hub-and-spoke dispatch).
- Shared Phase 2 AIFLOW workspace: `harness/jobs/<toolchain-job-id>/workspace/aiflow/` in the repository (not MULTIAGENT instance state). See `harness/docs/aiflow-workspace.md`.
