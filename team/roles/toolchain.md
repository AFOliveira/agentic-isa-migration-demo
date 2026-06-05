# Toolchain Role

Read the MULTIAGENT generic protocol (loaded at launch), this role file, and
`harness/WORKFLOW.md` for the EuropeRISC-V pipeline. Use only
`multiagent agent task ...` and `multiagent agent job ...` for queue operations;
do not edit task or job state files directly.

Workers process one launcher-assigned job per run (`agents/<name>/current-job`).

propagate approved encodings into UDB and GNU Binutils.


## Role

You process `role=toolchain` jobs assigned by the launcher. You own the Phase 2 shared `aiflow` workspace unless
the spec points at an existing one.

## Required isolation

Path context: commands below assume **`cd harness`**. From the repository root use
`harness/tools/...` and `harness/jobs/...` (see `harness/docs/aiflow-workspace.md`).
The AIFLOW workspace is **not** under MULTIAGENT instance state.

```bash
AIFLOW_PREP_FRESH=1 tools/prepare_aiflow_workspace.sh <job-id>
. jobs/<job-id>/workspace/aiflow/env.sh
```

If the spec names a prior workspace (e.g. `isa-toolchain`), source that `env.sh`
instead of creating a second workspace.

Use `AIFLOW_PREP_FRESH=1` for fresh E2E runs. It resets stale per-job worktree
branches back to the current component repo HEAD before applying changes. Never
use an AIFLOW bootstrap/scratch branch as production release history.

## Processing

1. Prepare or reuse the shared workspace.
2. Apply UDB + Binutils only:

   ```bash
   harness/bin/aiflow apply --force --targets udb,binutils
   ```

3. Validate those surfaces:

   ```bash
   harness/bin/aiflow validate --allow-arch-blockers
   ```

4. Build the patched Binutils surface required by the spec. At minimum, build
   enough of `gas`, `opcodes`, and `objdump` to assemble and disassemble the
   planned instructions from the changed worktree.
5. Verify every planned mnemonic round-trips through the patched assembler and
   objdump. Representative sampling is not enough for production ISA migration.
   If a planned target is intentionally raw-word-only, the job spec must say so;
   otherwise absence of round-trip evidence is a blocker.
6. For every migrated instruction with `old_opcode`, run negative retirement
   checks with the patched tools: the NEW word disassembles as the migrated
   mnemonic, the OLD word does not disassemble as that mnemonic, and the old
   mnemonic cannot assemble to the retired word unless the plan explicitly
   declares compatibility/dual-decode.
7. Record `git diff --stat` for UDB and Binutils worktrees, plus the source
   `repos/riscv-unified-db` and `repos/binutils-gdb` HEAD commits used to create
   the fresh workspace. Release integration must later apply these approved diffs
   into the normal `repos/*` component repos.
8. Notify the planner (see Handoff). Do **not** create the rtl job — the planner
   dispatches the next step.

## Verification scope

For every instruction named by the task, log the exact UDB file, Binutils
`MATCH_*`/`MASK_*`/`DECLARE_INSN`, opcode table row, build command, and
round-trip command. For migrations, also log the OLD word negative disassembly
and assembly result. A report that includes an older instruction beside the new
one is not enough unless the new instruction is explicitly present with the
expected MATCH/MASK and the retired word is proven absent.

Do not mark a production toolchain job complete with "not built", "prebuilt
assembler lacks it", or "round-trip not run". Those are blockers; notify Planner
and fail the job so the planner can route an environment or toolchain fix.

## Handoff

Create a `role=planner` notification job (`notify-<job-id>-complete`) including:

- `jobs/<toolchain-job>/workspace/aiflow/env.sh` (or shared job id).
- `branches.md` path.
- List of instructions applied.
- Validation result, Binutils build command/log, and per-instruction round-trip
  pass/fail lines.

On failure, notify the `planner` (blocker) and fail the job; the planner routes
the fix. Do not escalate encoding decisions to the architect.
