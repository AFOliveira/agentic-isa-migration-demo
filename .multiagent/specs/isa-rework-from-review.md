# Rework plan from reviewer rejection

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

The **reviewer** rejected a production gate (`isa-review` or
`isa-release-review`). Read the review job log and this spec's change list.
Re-queue the minimum fix jobs, then schedule the same review gate again when
dependencies are satisfied.

## Inputs

- Source review job: (filled by reviewer when creating this job)
- Task id: from the review job `task-id` file, when present

## Processing

1. Parse the reviewer’s change table (surface → role → paths).
2. Create targeted fix jobs for owning roles (`toolchain`, `rtl`, `simulator`,
   `zephyr`, `compliance`, `sysemu-run`, `verilator-run`, `integrator`) using
   `.multiagent/specs/isa-*.md` or inline specs. Each fix spec must name verification steps.
3. If integration bundle is stale after fixes, re-create `isa-integrate` and
   Phase 3 jobs (`isa-run-sysemu`, `isa-run-verilator`) as needed.
4. When all listed fixes are **done**, create the failed gate again. For the
   pre-release gate:

   ```bash
   multiagent agent job create isa-review -r reviewer -t <task-id> .multiagent/specs/isa-review.md
   ```

   For the final release gate:

   ```bash
   multiagent agent job create isa-release-review -r reviewer -t <task-id> .multiagent/specs/isa-release-review.md
   ```

5. `multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"` on this planner job with the job IDs created.

## When Done

Planner has re-queued work; reviewer will run again. Do **not** call
`multiagent agent task state <task-id> done` from here only when closing the overall task; do not create `isa-document` from this rework job.


## MULTIAGENT notes

- Job specs are passed as a file path to `multiagent agent job create`.
- Workers must create a `role=planner` notification job on the same task before `job done` / `job fail` / `job release` (hub-and-spoke dispatch).
- Shared Phase 2 AIFLOW workspace: `harness/jobs/<toolchain-job-id>/workspace/aiflow/` in the repository (not MULTIAGENT instance state). See `harness/docs/aiflow-workspace.md`.
