# Phase 2 — Zephyr runtime

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Verify or add Zephyr PS/PI support and build the migration sample. This is a
production gate: missing `west`, missing SDK, or missing board setup is a
blocker unless the human explicitly changes the task to dry-run scope.

## Workspace

Reference: `jobs/isa-toolchain/workspace/aiflow/env.sh`

Zephyr tree: `repos/zephyr` (do not edit trees outside `implementation/`).

## Processing

1. Align intrinsics/samples with the encoding plan.
2. Build the migration sample with `west build -b <board> <sample>`. Prefer the
   board/app from `<operator-board-app-repo>` or existing Zephyr
   project docs. If no board is obvious, inspect the Zephyr tree and pick the
   closest AIFoundry/minion board target; record the choice.
3. Prove whether the Zephyr artifact uses the migrated Binutils:
   - Preferred: build or assemble a Zephyr migration artifact through the patched
     assembler from `isa-toolchain`, using mnemonic emitters for the nine approved
     targets, then inspect it with the patched `objdump`.
   - If the normal `west build` uses only the stock Zephyr SDK and raw `.word`
     constants, record that honestly as compile-only fallback and create a
     blocker/follow-up for patched Binutils integration. Do not claim the
     Binutils port is used by Zephyr.
4. If the sample executes raw custom words or intrinsics on the `main()` path,
   provide boot/run evidence. If the sample is compile-only, keep raw-word
   emitters non-executed (`__used`/static noinline/linker-retained or equivalent)
   and record source/objdump evidence that `main()` does not call them. Inline
   asm `.word` helpers with immediate constraints must be macro/constant-only or
   guarded against runtime-variable arguments.
5. Attempt Zephyr runtime evidence:
   - Boot/run the built Zephyr ELF on `sys_emu` when the board/loader path exists;
     record console output, exit status, and exact command.
   - Discover whether a full-system Verilator target can boot this Zephyr board.
     If present, run it; if absent, record the missing target/path as an
     infrastructure blocker/gap. Decoder-level RTL cosim is not Zephyr runtime.
6. Required:

   ```bash
   AIFLOW_CONFIG=jobs/isa-toolchain/workspace/aiflow/repos.yaml \
     harness/bin/aiflow validate --allow-arch-blockers
   ```

Do not pass production Zephyr with stock-SDK static-only evidence. Static
encoding checks may be logged as debugging evidence, but the job is blocked
until `west build` passes, patched-Binutils usage is proven or explicitly
blocked, and runtime-executed custom instructions are backed by run evidence.

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
