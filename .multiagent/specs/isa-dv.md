# Phase 2 — directed DV (core-et)

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Author an **equivalence + retirement** test in core-et DV — independent of the
`rtl` author — proving the target NEW encoding decodes identically to its
baseline/original **reference** and that the OLD encoding is retired from the
migrated decoder. For migrations, use the old encoding only as a baseline
reference or recorded baseline control bundle; in the migrated decoder the old
word must be illegal or non-target unless compatibility is explicit.

## Workspace

```bash
. jobs/isa-toolchain/workspace/aiflow/env.sh
```

Author tests in the core-et RTL worktree named by the RTL job/MANIFEST (so the
test ships with the decode row and is captured by integration).

## Processing

1. Identify the **reference** from the spec: migration → the instruction at its
   old encoding (`old_opcode`) on baseline/original RTL or a saved baseline
   control bundle; new instruction → the named equivalent instruction (e.g.
   `addi`).
2. In `hw/ip/minion/frontend/dv/intpipe_decode_test.cc` (or the `vpu_decoder` TB
   for packed/VPU ops), expose the decode control fields, then capture the control
   for the target's NEW-encoding word and assert it equals the baseline reference
   control (+ class bits). Also drive the OLD word in the migrated decoder and
   assert it is illegal or does not match the migrated target, unless the plan
   explicitly declares compatibility/dual-decode. Add a separate negative
   control. Create `hw/ip/<block>/dv/` and/or `dv/rtlcosim/<module>/` per
   `repos/core-et/AGENTS.md` when applicable.
3. Run and record:

   ```bash
   make -C repos/core-et/hw/ip/minion/frontend/dv test   # TEST PASSED incl. new assertions
   make -C repos/core-et lint                            # zero warnings
   ```

4. If the spec-derived expectation ≠ the RTL decode, fix the module not the test:
   notify the planner with the defect and suggest `rtl` fix routing. Never weaken
   assertions.

## Acceptance (all required)

- The test drives the target NEW-encoding word and asserts identical decoded
  control versus the baseline reference (target ≡ baseline reference).
- For migrations, the test also drives the OLD word in the migrated decoder and
  asserts retirement (illegal or non-target), unless compatibility is explicit.
- A separate negative control fails.
- `make … dv test` prints `TEST PASSED` including the equivalence assertions; `lint` clean.

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
