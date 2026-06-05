# Final release review

## Task

`<task-id>` — set to the active MULTIAGENT task.

## Target

`<implementation-root>`

## Objective

Independently verify that `isa-release-integrate` produced a production-ready
multi-repo release. This gate decides whether documentation and task closure are
allowed.

## Prerequisites

- `isa-review` passed.
- `isa-release-integrate` is done.
- `artifacts/isa-release/MANIFEST.md` exists.

## Processing

1. Read the original task, `isa-review`, `isa-release-integrate`, and
   `artifacts/isa-release/MANIFEST.md`.
2. For every pushed component branch in the manifest, verify:
   - remote ref exists and points at the recorded commit;
   - branch contains the intended migration files;
   - branch has a merge-base with its declared remote release base and is not
     rooted at an AIFLOW `bootstrap` commit;
   - branch path is a normal component repo under `repos/*`, not only
     `harness/jobs/.../workspace/aiflow/repos/*`;
   - worktree is clean or only ignored build output remains.
3. Verify the implementation release branch records every touched component
   commit as a submodule pointer or otherwise names normal pushed refs that a
   fresh clone can fetch. A manifest that relies on absolute `harness/jobs/...`
   paths as source-of-truth fails release review.
4. Re-run or spot-rerun enough verification to trust the manifest:
   - `aiflow validate --allow-arch-blockers --allow-pending`
   - Binutils round-trip and OLD-word retirement for every migrated mnemonic
   - sysemu execute-equivalence 9/9 plus OLD-word rejection in migrated `sys_emu`
   - core-et frontend DV/lint/required cosim plus OLD-word retirement in migrated decode
   - Zephyr `west build` migration sample, proof whether the artifact used the
     patched Binutils or only stock Zephyr SDK raw-word fallback, boot/run
     evidence on `sys_emu` for runtime sample paths, full-system Verilator
     boot/run when a target exists, and non-execution proof for compile-only
     emitters
5. Build a final coverage matrix mapping every planned instruction to UDB,
   Binutils, sysemu, Zephyr, RTL/DV/cosim, OLD-word retirement evidence, and
   release branch/commit.
6. For every Verilator claim about architecturally visible state, register
   writeback, retirement, or destination-register value, inspect the harness
   source and add the reviewer proof-boundary matrix from `roles/reviewer.md`.
   A PASS label alone is not evidence. If the harness is module-level or
   testbench-owned, classify it as such and reject/block any broader
   architectural claim unless the task explicitly accepts that narrower scope.

## Pass

- All pushed refs exist and match the manifest.
- All release surfaces are normal component repo branches with sane history, not
  scratch AIFLOW worktree branches.
- The implementation release state is self-contained for a fresh clone through
  submodule pointers or normal pushed component refs.
- All required release checks pass.
- No release-relevant dirty worktree remains.
- Every migrated instruction has OLD-word retirement evidence on every touched
  migrated artifact unless compatibility/dual-decode is explicit.
- No production evidence was downgraded to static-only or skipped residuals.
- Any architectural-state claim has matching full-core/pipeline/register-file
  proof, or the release review records an explicit blocker instead of passing.

## When Done

- **Pass:** create `notify-isa-release-review-pass` for `planner` with the final
  coverage matrix. Planner creates `isa-document`.
- **Changes needed:** create `notify-isa-release-review-changes` for `planner`
  with a table: `surface | symptom | owning role | exact command/log`.
- **Blocked:** notify Planner and fail this job if the release cannot be
  verified.

Then run `multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"`.

## MULTIAGENT notes

- Job specs are passed as a file path to `multiagent agent job create`.
- Workers must create a `role=planner` notification job on the same task before `job done` / `job fail` / `job release` (hub-and-spoke dispatch).
- Shared Phase 2 AIFLOW workspace: `harness/jobs/<toolchain-job-id>/workspace/aiflow/` in the repository (not MULTIAGENT instance state). See `harness/docs/aiflow-workspace.md`.
