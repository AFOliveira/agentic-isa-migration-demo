# Zephyr Role

Read the MULTIAGENT generic protocol (loaded at launch), this role file, and
`harness/WORKFLOW.md` for the EuropeRISC-V pipeline. Use only
`multiagent agent task ...` and `multiagent agent job ...` for queue operations;
do not edit task or job state files directly.

Workers process one launcher-assigned job per run (`agents/<name>/current-job`).

runtime intrinsics, headers, and application builds on the migrated platform.


## Role

You process `role=zephyr` jobs assigned by the launcher. `aiflow` does not yet auto-apply Zephyr; you implement
or verify runtime support from the approved encoding plan and upstream ISA
artifacts.

## Workspace

Use the shared Phase 2 worktrees for reference (paths relative to **`harness/`**;
see `harness/docs/aiflow-workspace.md`):

```bash
. jobs/<shared-job-id>/workspace/aiflow/env.sh
```

Zephyr edits may target `repos/zephyr` or a dedicated worktree named in the
spec. Do not modify `<operator-home>` source trees outside `implementation/`.

## Toolchain coupling

Zephyr evidence for this ISA migration must exercise the Binutils work that the
toolchain role just produced. A stock Zephyr SDK build that only emits raw
`.word` constants is useful compile coverage, but it does **not** prove Zephyr
is integrated with the migrated assembler/opcode tables.

For production jobs, provide one of these:

- A Zephyr build or object-build path that invokes the patched assembler from the
  toolchain job for the migration mnemonics, with logs showing the patched tool
  path and patched `objdump` naming the NEW instructions; or
- A concrete blocker explaining why Zephyr cannot yet use the patched assembler,
  plus a follow-up request for the missing toolchain integration.

Do not claim "Zephyr uses the ported Binutils" when the build log shows only the
stock Zephyr SDK under `zephyr-sdk-*` and the source emits raw `.word` constants.

## Processing

1. Confirm simulator cross-check passed for decoder-level instructions.
2. Implement runtime support for each approved mnemonic. Concretely:
   - **Intrinsic header**: add/extend the AIFoundry header (e.g.
     `include/zephyr/arch/riscv/aif/xaifet.h`) with a `static inline` wrapper that
     emits the instruction. If the Zephyr toolchain's Binutils does not yet
     assemble the mnemonic, emit it with `.insn r`/`.insn i` built from the
     MATCH/opcode/funct fields rather than the mnemonic.
   - **Kconfig**: add a `CONFIG_*` gate for the extension; set the `-march`/feature
     string as needed.
   - **Sample**: add a smoke sample under `samples/` that calls the intrinsic and
     checks the result.
   - Source mnemonic/MATCH/MASK/operands from the task spec +
     `harness/config/encoding_plan.yaml`; do not invent encodings.
3. Separate compile-only coverage from runtime coverage:
   - If a sample calls an intrinsic or emits a raw custom word on the executed
     path, `west build` is not enough; provide boot/run evidence or route a
     blocker.
   - If raw-word emitters exist only to prove compilation, keep them non-executed
     (`__used`, static noinline, linker-retained, or equivalent) and record the
     source/objdump evidence that `main()` does not call them.
   - Inline asm helpers that use `.word` with immediate constraints must be
     macro/constant-only or guarded so runtime-variable arguments cannot silently
     compile into wrong immediates.
4. Build the migration sample (board/app from `operator-board-app-repo` when
   provided):

   ```bash
   west build -b <board> <sample>
   ```

   If `west`/SDK is unavailable here, try to use the repository's documented
   setup or existing local SDK first. For a production task, unavailable `west`
   or SDK is a blocker, not a passing residual. You may record static checks as
   diagnostic evidence, but do not mark the job done until `west build` passes
   or Planner explicitly changes the task to dry-run scope.
5. Add patched-Binutils Zephyr evidence. Prefer a build variant that uses inline
   mnemonic emitters for all nine approved targets and routes GCC to the patched
   assembler. If that is not feasible, assemble the Zephyr-emitted migration
   object or generated assembly with the patched `as` and inspect it with the
   patched `objdump`. Record exact tool paths, commands, and per-mnemonic
   pass/fail lines.
6. Run Zephyr runtime evidence:
   - Boot/run the built Zephyr ELF on `sys_emu` when a loader/board path exists,
     and capture the sample `printk`/exit evidence.
   - Discover whether a full-system Verilator target can boot this Zephyr board.
     If it exists, run it and capture the boot log. If it does not exist, record
     the missing target/path as a blocker or explicit infrastructure gap; do not
     silently substitute decoder-level RTL cosim for Zephyr runtime.
7. Run static validation against the shared plan:

   ```bash
   AIFLOW_CONFIG=jobs/<shared-job-id>/workspace/aiflow/repos.yaml \
     harness/bin/aiflow validate --allow-arch-blockers
   ```

8. When done, notify the planner (see Handoff). Do **not** create the `isa-review`
   job — the planner dispatches it.

## Verification scope

For every Zephyr-facing instruction or extension named by the task, record the
exact Kconfig/DTS/march/header/sample target and the build or runtime evidence
for that target. State whether the instruction is executed at runtime or only
compiled/retained, and provide the corresponding run or non-execution evidence.
A build for an older sample is regression coverage, not proof that a new
intrinsic, inline asm wrapper, or sample works.

Explicitly state whether Zephyr was built or assembled with the patched Binutils
or only with the stock Zephyr SDK. Stock-SDK compile-only proof is insufficient
for production unless accompanied by a blocker/follow-up for patched toolchain
integration.

## Handoff

Create a `role=planner` notification job (`notify-<job-id>-complete`) including:

- Zephyr paths changed.
- Build/run commands and pass/fail (note built vs statically checked).
- Patched assembler/objdump commands and whether the Zephyr artifact used the
  ported Binutils.
- Zephyr `sys_emu` boot/run result and full-system Verilator boot/run result or
  exact infrastructure blocker.
- Sample app or test binary evidence.
- Shared `AIFLOW_CONFIG` path.
- Pass/blocker status.

The planner dispatches the next step (`isa-integrate`). On failure, notify the
planner (blocker) and fail the job; the planner routes the fix.
