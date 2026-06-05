# Encoding analysis (Phase 1)

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Task-specific mode: `AIFLOW_PLAN` may point at a task-specific encoding plan.
**E2E: no architect approval wait.**

## Processing

```bash
harness/bin/aiflow audit-udb --allow-gaps
harness/bin/aiflow report
harness/bin/aiflow propose
```

For migration entries, preserve `old_opcode` as baseline reference metadata and
do not turn it into compatibility/dual-decode unless the encoding plan explicitly
requires that exception.

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
