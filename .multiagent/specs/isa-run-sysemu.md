# Phase 3 — run runtime smoke on sys_emu (sw-sysemu)

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Run the **execute-equivalence differential** for the target under the functional
ET-SoC1 emulator: the instruction at its NEW encoding must produce the same
architectural result as its **reference** (old encoding for a migration / the
named equivalent instruction for a new one). The simulator job supplies the
equivalence vector (target word, reference word, operands, expected result).

## Processing

From the **repository root** (MULTIAGENT agents use the repo checkout as cwd):

```bash
export AIFLOW_RUNTIME_DIR=harness/runtime/<task>     # has the equivalence ELF(s)
harness/tools/phase3-runtime.sh sysemu
```

Run **both** sides and diff architectural state (rd / fflags / memory; the full
output vector for packed-SIMD):
- target NEW-encoding word on the migrated `sys_emu`;
- OLD word on the migrated `sys_emu` as a negative retirement check for
  migrations (must reject/trap/not dispatch unless compatibility is explicit);
- reference word — old encoding on the **baseline** `sys_emu` (migration), or the
  equivalent instruction (new).

## Acceptance (all required)

- Both runs exit **0**.
- The architectural diff (target vs baseline reference) is **empty**.
- The migrated OLD-word negative check is recorded and passes for migrations.
- Target, baseline reference, OLD-negative logs, and the diff are recorded.
  (Smoke-only is acceptable ONLY if the spec explicitly scopes it to a smoke.)

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
