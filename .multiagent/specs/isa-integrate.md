# Integrate reviewed migration

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Package or merge reviewed work from `isa-toolchain` worktrees per the review job.

## Processing

1. Confirm Phase 2 apply jobs (toolchain → zephyr) completed.
2. Produce integration artifact (patch bundle under `artifacts/` or merge
   instructions) as named in the review spec.
3. Include the ISA Migration Release Contract evidence in the artifact: per
   migrated instruction, NEW-encoding acceptance, baseline-reference equivalence,
   and OLD-encoding retirement/rejection on each touched migrated surface.
4. Log final branch manifest and validation summary.

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
