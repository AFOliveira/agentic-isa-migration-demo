# Simulator Role

Read the MULTIAGENT generic protocol (loaded at launch), this role file, and
`harness/WORKFLOW.md` for the EuropeRISC-V pipeline. Use only
`multiagent agent task ...` and `multiagent agent job ...` for queue operations;
do not edit task or job state files directly.

Workers process one launcher-assigned job per run (`agents/<name>/current-job`).

update `et-platform` sysemu decoder dispatch and cross-check against RTL.


## Role

You process `role=simulator` jobs assigned by the launcher. You apply sysemu changes and perform the RTL ↔
simulator cross-check from the poster.

## Workspace

Paths relative to **`harness/`** (`harness/docs/aiflow-workspace.md`).

```bash
. jobs/<shared-job-id>/workspace/aiflow/env.sh
```

## Processing

1. Confirm the RTL and DV jobs completed (decode row generated; directed DV test
   authored and passing).
2. Update the et-platform sysemu decoder for each instruction the task names.
   `aiflow apply --targets sysemu` only **moves an existing** handler between
   decode groups — it does **not** create a handler for a brand-new instruction:
   - Existing-instruction remap/move:
     `harness/bin/aiflow apply --force --targets sysemu`.
   - **New instruction — author by hand** (mirror the neighbouring instructions in
     each file):
     - declare it in `repos/et-platform/sw-sysemu/insn_func.h`
       (`void insn_<mnemonic>(...)`),
     - define it in the matching `repos/et-platform/sw-sysemu/insns/<group>.cpp`
       (integer ALU ops live in `arith.cpp`); an ADDI-equivalent body is
       `WRITE_RD(RS1 + IIMM);`,
     - add dispatch in `repos/et-platform/sw-sysemu/processor.cpp` in the decode
       function for the opcode (e.g. `dec_custom1`, `case 0x4:` for custom-1
       `funct3=100`) returning `insn_<mnemonic>`.

3. Validate decoder consistency:

   ```bash
   harness/bin/aiflow validate --allow-arch-blockers
   ```

4. Cross-check: for each migrated instruction in the spec, confirm the sysemu
   `dec_*` path matches the core-et decode case (the `32'b…: raw_inst_ctrl = {…}`
   row in `repos/core-et/.../intpipe_decode.sv`) and Binutils MATCH/MASK. Compare
   trace or log output with core-et DV when commands are provided.
5. For every migrated instruction with `old_opcode`, confirm the migrated
   `sys_emu` decoder no longer dispatches the OLD word to the migrated handler
   unless compatibility/dual-decode is explicit. The old word may appear only in
   the baseline reference program.
6. **Author the execute-equivalence vector** (the proof the migration preserved
   behavior). The handler body is unchanged — only its dispatch encoding moved —
   so the target at its NEW encoding must produce the SAME architectural result as
   its **baseline reference** (the instruction at its `old_opcode` on baseline
   `sys_emu` for a migration, or the named equivalent instruction for a new one).
   Build a tiny **raw-word ELF** (no
   assembler needed — emit the 32-bit words directly, as the runtime harness's ELF
   generator does) that loads fixed operands and runs the instruction, in two
   forms — the target NEW-encoding word and the reference word — and define the
   architectural result to capture (rd / fflags / memory; the **full output
   vector** for packed-SIMD). Place it in the task runtime harness so `sysemu-run`
   can execute both and diff. Record operands + expected (reference) result.
7. If RTL and sysemu disagree, do **not** create a fix job yourself — notify the
   planner (blocker) with the exact mismatch; the planner routes the fix.
8. When done, notify the planner (see Handoff). Do **not** create the `zephyr`
   job — the planner dispatches the next step.

## Verification scope

For each instruction named in the task, prove the exact sysemu decoder and
architectural effect, or record the blocker. For migrations, also prove the OLD
word is not live in migrated `sys_emu`. A trace for one mnemonic does not prove
another mnemonic, even if their encodings are adjacent.

When sysemu is the only executable proof, say that plainly in the handoff. Do
not imply that sysemu success also proves Verilator, RTL simulation, Zephyr, or
hardware behavior unless those commands were run for the same target.

## Handoff

Create a `role=planner` notification job (`notify-<job-id>-complete`) including:

- Shared `env.sh` path and the sysemu worktree/branch.
- Cross-check matrix (instruction → core-et decode case (`raw_inst_ctrl` row) → sysemu branch → pass/fail).
- The **execute-equivalence vector**: the test program path, the target NEW-encoding
  word, the baseline reference word, the operands, and the expected result — so
  `sysemu-run` can run both and diff in Phase 3.
- Commands run and logs.
- Pass/blocker status.

The planner reads this and dispatches the `zephyr` job. Do not create it yourself.
