# Phase 2 — simulator (sysemu)

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Apply/author the sysemu decoder change, cross-check against the core-et decode
case, and author the **execute-equivalence vector** (test program + target
NEW-encoding word + reference word + operands + expected result) that `sysemu-run`
runs as the migration proof.

## Workspace

```bash
. jobs/isa-toolchain/workspace/aiflow/env.sh
```

## Processing

```bash
harness/bin/aiflow apply --force --targets sysemu
harness/bin/aiflow validate --allow-arch-blockers
```

Cross-check each migrated instruction: the core-et decode case
(`32'b…: raw_inst_ctrl = {…}` in `intpipe_decode.sv`) vs the sysemu `dec_*` path.
For migrations with `old_opcode`, also prove the OLD word no longer dispatches
to the migrated handler in migrated `sys_emu` unless compatibility/dual-decode is
explicit.

Author the **execute-equivalence vector** for `sysemu-run`: a raw-word ELF (no
assembler needed) that runs the instruction with fixed operands in two forms —
target NEW-encoding word and reference word (reference = the instruction at its
`old_opcode` on baseline `sys_emu` for a migration, or the named equivalent
instruction for a new one). Include a migrated-`sys_emu` OLD-word negative case.
Place it under the task runtime harness; record operands + expected (reference)
result. A migration leaves the handler body unchanged (only dispatch moves), so
target and reference must produce identical architectural state.

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
