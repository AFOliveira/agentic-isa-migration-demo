# Phase 2 — toolchain (UDB + Binutils)

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Own the shared Phase 2 workspace. Apply UDB and Binutils encodings.

## Processing

```bash
AIFLOW_PREP_FRESH=1 harness/tools/prepare_aiflow_workspace.sh isa-toolchain
. harness/jobs/isa-toolchain/workspace/aiflow/env.sh
harness/bin/aiflow apply --force --targets udb,binutils
harness/bin/aiflow validate --allow-arch-blockers
```

`AIFLOW_PREP_FRESH=1` is required for fresh E2E runs. It resets stale
`harness/isa-toolchain/*` worktree branches back to the current component repo
HEAD before applying changes. Do not reuse a previous run's bootstrap or scratch
branch as release history.

Build patched Binutils from the changed worktree enough to produce patched
`gas`/`objdump`, then round-trip every planned mnemonic through assemble and
disassemble. Record exact commands/logs. If a target is intentionally raw-word
only, name it and explain why; otherwise missing round-trip evidence is a
blocker.

For every migrated instruction with `old_opcode`, also record negative
retirement evidence from the patched tools: NEW word disassembles as the migrated
mnemonic; OLD word does not disassemble as the migrated mnemonic; old mnemonic
does not assemble to the retired word unless compatibility/dual-decode is
explicit in `encoding_plan.yaml`.

Record diffs for UDB and Binutils worktrees. Also record the source repo HEADs
under `repos/riscv-unified-db` and `repos/binutils-gdb`; release integration
must later materialize these diffs into those normal component repos, not push
the scratch AIFLOW worktrees directly.

## When Done

Notify the planner: create a `planner` notification (`notify-<job-id>-complete`)
with your results, artifact pointers (shared `env.sh`, worktree/branch, files
produced), and pass/blocker status. Do **not** create the next role's job — the
planner authors and dispatches the next step (see `roles/planner.md` -> Dispatch
model). On failure, notify the planner as a blocker and fail the job.


## MULTIAGENT notes

- Job specs are passed as a file path to `multiagent agent job create`.
- Workers must create a `role=planner` notification job on the same task before `job done` / `job fail` / `job release` (hub-and-spoke dispatch).
- Shared Phase 2 AIFLOW workspace: `harness/jobs/<toolchain-job-id>/workspace/aiflow/` in the repository (not MULTIAGENT instance state). See `harness/docs/aiflow-workspace.md`.
