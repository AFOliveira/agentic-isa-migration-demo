# Phase 2 — match/mask manifest

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Decodability gate + baseline manifest, before any repo write.

## Processing

1. **Decodability gate (must pass first):** run UDB's canonical encoding-conflict
   check on the proposed encodings —

   ```bash
   AIFLOW_PLAN=<proposed encoding_plan.yaml> harness/bin/check-encodings
   ```

   Exit 0 = uniquely decodable. **Non-zero = conflict** (e.g. an opcode-only
   catch-all shadowing siblings): do not proceed; notify the planner as a blocker
   with the conflicting pairs + sample words so it bounces back to
   `encoding-analyst`. No `--allow` override exists for a decode conflict.

2. Only if the gate passes, record the manifest:

   ```bash
   harness/bin/aiflow report
   harness/bin/aiflow propose
   harness/bin/aiflow validate --allow-pending
   ```

   Log mnemonic → new_match → MATCH/MASK for each planned instruction. For
   migrations, also log `old_opcode` as the baseline reference and state whether
   compatibility/dual-decode is explicit; otherwise the OLD encoding is retired.

## When Done

- **Gate PASS:** `planner` notification (`notify-<job-id>-complete`) with the
  manifest, OLD-reference/retirement rows, the `check-encodings` PASS line, and
  artifact pointers.
- **Gate FAIL:** `planner` notification (`notify-<job-id>-conflict`, blocker) with
  every conflicting pair + sample word + the offending instruction; fail the job.
  Do **not** create the next role's job — the planner dispatches per
  `roles/planner.md` (Phase-1 decodability loop).


## MULTIAGENT notes

- Job specs are passed as a file path to `multiagent agent job create`.
- Workers must create a `role=planner` notification job on the same task before `job done` / `job fail` / `job release` (hub-and-spoke dispatch).
- Shared Phase 2 AIFLOW workspace: `harness/jobs/<toolchain-job-id>/workspace/aiflow/` in the repository (not MULTIAGENT instance state). See `harness/docs/aiflow-workspace.md`.
