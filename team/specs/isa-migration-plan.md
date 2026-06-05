# Plan AIFoundry ISA Migration (Production E2E)

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

Production workloads run on `<remote-user>@<remote-workload-host>` under
`<implementation-root>`. The local checkout is the
control surface for MULTIAGENT state and prompt edits.

## Objective

Run the full production E2E flow. Proceed with `--force` / `approved: true` for
entries already marked approved in `encoding_plan.yaml`, but do not treat
`needs_arch_decision` entries as implemented. The task is not complete until all
touched repos have committed and pushed release branches, required builds/tests
pass from those branches, final release review passes, and documentation records
the release state.

Fresh E2E runs must create the Phase 2 workspace with `AIFLOW_PREP_FRESH=1` and
must not reuse stale `harness/isa-toolchain/*` scratch branches. The final
release must be self-contained through normal component repos under `repos/*`;
`harness/jobs/.../workspace/aiflow/repos/*` is scratch input only.

Enforce `harness/WORKFLOW.md` -> ISA Migration Release Contract throughout the
plan. For every `status: proposed` entry with `old_opcode`, each job spec must
require both positive NEW-encoding evidence and negative OLD-encoding retirement
evidence on the migrated artifacts, unless compatibility/dual-decode is
explicit in `encoding_plan.yaml`. Zephyr build-only evidence does not prove a
sample that executes raw custom instructions; require boot/run evidence or prove
compile-only emitters are non-executed.

## When Done

Hub-and-spoke: **you (planner) create every pipeline job** and dispatch the next
when its dependencies are done — agents do not chain to each other (the one
exception is rtl→dv, a sub-chain). Create the first job only; each agent's
completion notification is your cue to create the next (see
`roles/planner.md` → Dispatch model).

```bash
multiagent agent job create isa-udb-audit -r udb-auditor -t <task-id> .multiagent/specs/isa-udb-audit.md
```

Dependency order you dispatch (production gate):

```text
isa-udb-audit -> isa-encoding-analyst -> isa-compliance -> isa-toolchain
  -> isa-rtl --(sub-chain)--> isa-dv        # rtl creates isa-dv directly
  -> isa-simulator -> isa-zephyr -> isa-integrate
  -> isa-run-sysemu + isa-run-verilator     # both, in parallel
  -> isa-review                             # reject -> route fixes -> re-create isa-review
  -> isa-release-integrate                  # committer: commit/push touched repos + release manifest
  -> isa-release-review                     # final gate: verify pushed clean release state
  -> isa-document
```

Use `.multiagent/specs/isa-*.md` as templates for the WHAT; agents supply HOW from their role
files. Complete this plan job after `isa-udb-audit` exists.


## MULTIAGENT notes

- Job specs are passed as a file path to `multiagent agent job create`.
- Workers must create a `role=planner` notification job on the same task before `job done` / `job fail` / `job release` (hub-and-spoke dispatch).
- Shared Phase 2 AIFLOW workspace: `harness/jobs/<toolchain-job-id>/workspace/aiflow/` in the repository (not MULTIAGENT instance state). See `harness/docs/aiflow-workspace.md`.
