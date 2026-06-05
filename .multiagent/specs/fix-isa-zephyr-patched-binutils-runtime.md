# Fix ISA Zephyr Patched-Binutils Runtime Gate

## Task

`isa-migration-clean-release-20260604`

## Context

The previous `isa-zephyr` job produced useful Zephyr compile evidence, but it is
not enough for production release:

- The `west build` used the stock Zephyr SDK.
- The migration sample used raw `.word` constants rather than assembler
  mnemonics emitted by the ported Binutils.
- The proof did not boot/run the Zephyr ELF on `sys_emu`.
- The Verilator evidence was decoder/cosim evidence, not a full-system Zephyr
  boot/run target.

The Zephyr role contract has been tightened. Treat stock-SDK/raw-word
compile-only evidence as a fallback diagnostic, not as a production pass.

## Inputs

- Implementation root: `<implementation-root>`
- Main branch under review: `multiagent/clean-release-flow-20260604`
- Zephyr tree: `repos/zephyr`
- Zephyr branch to inspect/update:
  `multiagent/isa-migration-clean-release-20260604/isa-migration-release`
- Binutils tree: `repos/binutils-gdb`
- Binutils branch/commit:
  `multiagent/isa-migration-clean-release-20260604/isa-migration-release`
  at `42f76cd94338df497f1ef5b0ddb029873e63fdad`
- Patched assembler:
  `harness/jobs/isa-toolchain/workspace/aiflow/repos/binutils-gdb/build-riscv64-unknown-elf/gas/as-new`
- Patched objdump:
  `harness/jobs/isa-toolchain/workspace/aiflow/repos/binutils-gdb/build-riscv64-unknown-elf/binutils/objdump`
- Previous Zephyr evidence:
  `artifacts/isa-integrate/zephyr-evidence.md`,
  `harness/jobs/isa-release-integrate-rerun-1/workspace/zephyr-west-build.log`,
  and `harness/jobs/isa-release-integrate-rerun-1/workspace/zephyr-compile-only-proof.log`

## Required Work

1. Re-check the Zephyr sample/header changes on the release branch.
2. Determine whether `west build` can be configured to invoke the patched
   assembler for the migration mnemonics.
3. Preferred proof: add or adjust a Zephyr migration artifact that uses mnemonic
   emitters for all nine approved migration targets, route assembly through the
   patched `as-new`, and inspect the object/ELF with the patched `objdump`.
   Record exact commands, tool paths, object paths, and per-mnemonic PASS/FAIL.
4. If a normal Zephyr build cannot use the patched assembler because only
   scratch `gas`/`objdump` artifacts exist, the Zephyr SDK cannot be pointed at a
   complete patched prefix, or required build dependencies are missing, record
   that exact blocker. Do not mark production Zephyr as passing.
5. Attempt Zephyr runtime evidence:
   - Boot/run the built Zephyr ELF on `sys_emu` if a board/loader path exists in
     `repos/et-platform`, `harness/runtime`, or the Zephyr board files. Capture
     command, console/log output, and exit status.
   - Discover whether a full-system Verilator target can boot the same Zephyr
     board/ELF. If it exists, run it. If it does not, record the exact missing
     target/path as an infrastructure blocker/gap.
   - Do not substitute decoder-level RTL cosim for Zephyr runtime.
6. Preserve compile-only fallback evidence if needed, but ensure any raw `.word`
   emitters are non-executed from `main()` and guarded against runtime-variable
   immediate arguments.
7. Update release evidence or manifest text only when the new evidence changes
   release status. If markdown changes, run `harness/tools/doc-audit`.

## Completion Criteria

Create one planner notification on this same task:

- `notify-fix-isa-zephyr-patched-binutils-runtime-complete` only if patched
  Binutils usage is proven and runtime requirements are satisfied or explicitly
  non-executing.
- `notify-fix-isa-zephyr-patched-binutils-runtime-blocked` if patched Binutils
  integration or Zephyr runtime infrastructure is missing.

Then mark this job done or failed consistently with that notification. Do not
push to GitHub from the server.
