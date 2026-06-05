# Release integrate ISA migration

## Task

`<task-id>` — set to the active MULTIAGENT task.

## Target

`<implementation-root>`

## Objective

Convert the reviewed migration work into a production release state. Commit and
push every touched component repo branch, verify from the committed branch state,
and write a release manifest. This is not a documentation-only commit.

## Sources

- `$MULTIAGENT_STATE_DIR/tasks/<task-id>/spec.md`
- `$MULTIAGENT_STATE_DIR/jobs/isa-review/log.md`
- `artifacts/isa-integrate/MANIFEST.md`
- `harness/jobs/isa-toolchain/workspace/aiflow/branches.md`
- `harness/jobs/isa-toolchain/workspace/aiflow/repos.yaml`
- `harness/config/encoding_plan.yaml`
- Normal component repos under `repos/*`; these are the release surfaces.

## Required Surfaces

For every surface touched by the migration, create or reuse a named release
branch in the normal component checkout under `repos/*`, commit only intended
source/test/doc changes, and push it to `origin`. The AIFLOW workspace is scratch
input only. Do not push `harness/jobs/.../workspace/aiflow/repos/*` branches as
production release branches.

Expected surfaces for the ISA migration are:

| Surface | Repo path | Required release verification |
|---------|-----------|-------------------------------|
| UDB | `repos/riscv-unified-db` | `aiflow validate` includes UDB entries |
| Binutils | `repos/binutils-gdb` | patched build plus assembler/objdump round-trip and OLD-word retirement evidence for every planned mnemonic |
| RTL CPU baseline mirror | `<baseline-rtl-repo>` when separately touched | release commit or explicit baseline-only note |
| sysemu | `repos/et-platform` | clean `sys_emu` build, execute-equivalence 9/9, and OLD-word rejection in migrated `sys_emu` |
| core-et | `repos/core-et` | frontend DV, lint, required migration cosim, and OLD-word retirement in migrated decode pass |
| Zephyr | `repos/zephyr` | `west build` of the migration sample passes; patched-Binutils usage is proven or recorded as a blocker; `sys_emu` boot/run evidence exists for runtime sample paths; full-system Verilator boot/run is attempted when a target exists; compile-only emitters are proven non-executed |
| implementation | repo root | release manifest committed if changed |

Do not commit generated dependency caches, build trees, transient logs, or
unrelated local files.

## Processing

1. Inspect all touched repos with `git status --short --branch`.
2. Create release branches using a stable name such as
   `harness/<task-id>/isa-migration-release`.
3. Materialize the approved migration changes from the reviewed AIFLOW/core-et
   worktrees into the normal `repos/*` component repos. Preferred methods are
   `git diff -- <paths> | git -C repos/<component> apply` or file-level copy for
   generated source files, followed by `git diff` review. Do not carry over
   generated build trees, vendor caches, transient reports, or absolute-path
   workspace files.
4. Before committing, verify each component release branch has a sane merge-base
   with its normal remote branch or documented release base. If there is no
   merge-base, or the branch root is an AIFLOW `bootstrap` commit, stop and report
   a release blocker instead of pushing.
5. Stage and commit only the approved migration changes per repo.
6. Update implementation submodule pointers for every touched component repo.
   A fresh clone of the implementation release branch must reproduce the release
   state after `git submodule update --init --recursive` and normal dependency
   builds. `artifacts/isa-integrate/env.sh` may be recorded as evidence, but it
   must not be the only source of release truth.
7. Push every release branch to `origin`.
8. Re-run release verification from the committed branch state:
   - `harness/bin/aiflow validate --allow-arch-blockers --allow-pending`
   - patched Binutils build and per-mnemonic assembler/objdump round-trip plus
     OLD-word negative disassembly/assembly checks
   - `python3 harness/runtime/isa-migration/gen_vectors.py`
   - `python3 harness/runtime/isa-migration/run_equiv_differential_strict.py`
     when present; otherwise the task-specific execute-equivalence command must
     prove both target/reference equivalence and migrated OLD-word rejection
   - `harness/tools/phase3-runtime.sh sysemu`
   - `make -C repos/core-et/hw/ip/minion/frontend/dv test`
   - `make -C repos/core-et lint`
   - required `repos/core-et/dv/rtlcosim/...` migration cosim with `ORIG_ROOT`
   - OLD-word retirement assertions in core-et DV and migrated `sys_emu`
   - `west build -b <board> <sample>` for the Zephyr migration sample, patched
     assembler/objdump evidence for Zephyr migration artifacts or a blocker,
     `sys_emu` boot/run evidence for runtime sample paths, full-system Verilator
     boot/run when available, and compile-only non-execution evidence as required
9. Write `artifacts/isa-release/MANIFEST.md` with branch names, commit hashes,
   pushed refs, verification commands, log paths, and any blockers.
   The manifest must name normal `repos/*` component branches and implementation
   submodule pointers; absolute `harness/jobs/...` paths can appear only as
   pre-release evidence pointers.
10. Confirm release-relevant worktrees are clean after excluding ignored build
   outputs.

## Acceptance

- Every touched component repo has a local commit and pushed remote branch.
- The implementation branch records every touched component commit as a submodule
  pointer or otherwise documents a normal remote branch that a fresh clone can
  fetch without AIFLOW scratch worktrees.
- No release component branch is based on an AIFLOW `bootstrap` root or lacks a
  merge-base with its declared release base.
- Required release verification passes; skipped build/test steps are blockers.
- No release-relevant dirty worktrees remain.
- `artifacts/isa-release/MANIFEST.md` exists and is accurate.

## When Done

Create `notify-isa-release-integrate-complete` as a `role=planner` notification
with the manifest path, pushed refs, verification summary, and pass/blocker
status. On any blocker, notify Planner and fail the job.

Then run `multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"`.

## MULTIAGENT notes

- Job specs are passed as a file path to `multiagent agent job create`.
- Workers must create a `role=planner` notification job on the same task before `job done` / `job fail` / `job release` (hub-and-spoke dispatch).
- Shared Phase 2 AIFLOW workspace: `harness/jobs/<toolchain-job-id>/workspace/aiflow/` in the repository (not MULTIAGENT instance state). See `harness/docs/aiflow-workspace.md`.
