# Phase 3 — run runtime smoke on Verilator / RTL

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Run core-et's native Verilator DV for the decoder block (and cosim/lint) covering
the task's RTL targets.

## Processing

From the **repository root**:

```bash
harness/tools/phase3-runtime.sh verilator
```

The wrapper sources `harness/runtime/env.defaults.sh` (Verilator on PATH) and runs
core-et make:

```bash
make -C repos/core-et/hw/ip/minion/frontend/dv test   # decoder block DV
make -C repos/core-et lint                             # zero warnings
make -C repos/core-et/dv/rtlcosim test                # cosim (needs ORIG_ROOT)
```

Set `ORIG_ROOT` to the original CORE-ET RTL tree to enable cosim — the **RTL
equivalence** proof (migrated module vs original, cycle-by-cycle, identical
stimulus), the RTL analogue of the sysemu execute-equivalence diff. In production
scope, missing `ORIG_ROOT` is a blocker. Find/create a baseline worktree instead
of skipping cosim.

## Acceptance (all required)

- `make` exits **0**.
- The core-et decoder block DV prints **`TEST PASSED`** for the task's target(s),
  including the `dv`-authored equivalence test (target NEW decode matches
  baseline reference control).
- For migrations, the same DV run proves the OLD word is retired in the migrated
  decoder (illegal or non-target) unless compatibility/dual-decode is explicit.
- `make -C repos/core-et lint` is clean.
- rtlcosim equivalence passes with `ORIG_ROOT` set.

Lint-only PASS is **not** sufficient.

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
