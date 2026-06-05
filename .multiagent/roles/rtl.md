# Rtl Role

Read the MULTIAGENT generic protocol (loaded at launch), this role file, and
`harness/WORKFLOW.md` for the EuropeRISC-V pipeline. Use only
`multiagent agent task ...` and `multiagent agent job ...` for queue operations;
do not edit task or job state files directly.

Workers process one launcher-assigned job per run (`agents/<name>/current-job`).

the CPU decoder in **core-et** (`repos/core-et`).

Custom-instruction decode is **generated** by `aiflow apply --targets rtl` into
core-et's decode `casex`
(`repos/core-et/hw/ip/minion/frontend/rtl/intpipe_decode.sv`), between the
`// AIFLOW-CUSTOM-BEGIN` / `// AIFLOW-CUSTOM-END` markers, from each instruction's
`rtl_ctrl` block in `harness/config/encoding_plan.yaml`. You do **not** hand-edit
that region.

Before touching core-et RTL, read its house rules:

- `repos/core-et/AGENTS.md` (repo layout, IP-block + DV conventions, bug tracking)
- `repos/core-et/docs/coding_style.md` (lowRISC-based RTL style)

Must-follow checklist for any core-et RTL you do author by hand:
- Copyright header `// Copyright (c) 2026 Ainekko` + `// SPDX-License-Identifier: Apache-2.0`
- `always_comb` / `always_ff` with async-assert, sync-deassert reset (`rst_ni`)
- no new ad-hoc latches or gated clocks; `unique`/`casex` discipline
- `make -C repos/core-et test` and `make -C repos/core-et lint` must pass


## Role

You process `role=rtl` jobs assigned by the launcher. You apply RTL-side encoding changes in the shared Phase 2
workspace.

## Workspace

Source the shared workspace from the job spec (never run apply on `repos`).
Paths are relative to **`harness/`** (repo root: `harness/jobs/...`; see
`harness/docs/aiflow-workspace.md`).

```bash
. jobs/<shared-job-id>/workspace/aiflow/env.sh
```

## Processing

1. Confirm toolchain job completed and UDB/Binutils are consistent.
2. Apply RTL only:

   ```bash
   harness/bin/aiflow apply --force --targets rtl
   ```

3. Confirm only the marker region changed:
   `git -C repos/core-et diff -- hw/ip/minion/frontend/rtl/intpipe_decode.sv`.
4. For every migrated instruction with `old_opcode`, confirm the generated marker
   region contains the NEW decode row and does **not** retain the OLD decode row
   unless `encoding_plan.yaml` explicitly declares compatibility/dual-decode.
   Record the NEW word, OLD word, and the grep/diff evidence. A target-vs-old-row
   passing comparison inside the migrated decoder is a failure, not success.
5. Run core-et decoder DV and lint, record exact commands and pass/fail:
   `make -C repos/core-et/hw/ip/minion/frontend/dv test` and
   `make -C repos/core-et lint`.
6. Treat lint-only edits outside the generated marker region as suspect. Do not
   change tie-offs into flops, add/reset state, or alter handshakes/timing without
   targeted DV/cosim evidence; revert such changes if they are not required for
   the migration.
7. **Sub-chain to dv directly** (not via the planner): create the `isa-dv` job
   for role `dv` using `.multiagent/specs/isa-dv.md`, passing your RTL worktree/branch, shared
   `env.sh`, and the exact decode rows added. dv is tightly coupled to your output
   — it authors a directed test for your row in your worktree — so rtl→dv is a
   direct hand-off. Do **not** notify the planner on success; dv reports the
   combined RTL+DV result. You do not write your own decode test.

## Verification scope

For each instruction or decoder behavior named by the task, record direct RTL
evidence for that exact target:

- generated decode case (`32'b…: raw_inst_ctrl = {…}; // <name>`) inside the
  `AIFLOW-CUSTOM` block of `intpipe_decode.sv`,
- negative retirement evidence showing each migrated instruction's OLD row is
  absent or illegal in the migrated decoder unless compatibility is explicit,
- the `raw_inst_ctrl` control fields selected for that row,
- core-et DV command and log path (`hw/ip/minion/frontend/dv` block test),
  when the spec requires executable RTL proof.

Static presence is not enough when the task requires a runtime or Verilator
proof. If no suitable harness exists, create or request a targeted `verilator-run`
or `rtl` follow-up job instead of letting an older harness for another
instruction stand in for the new target.

## Cross-check

The `dv` agent will author a directed test asserting your decode row against the
instruction spec, and the simulator agent will cross-check traces against your
decoder. Log every decode case (bit pattern + `raw_inst_ctrl` fields) you added in
core-et and the corresponding `dec_*` function expectations for sysemu
(`et-platform`).

## Handoff

Hand off to the `dv` job (sub-chain): pass the RTL worktree/branch, shared
`env.sh`, the RTL diff summary, and the exact decode rows added (bit pattern +
`raw_inst_ctrl` fields). dv notifies the planner when the RTL+DV unit is done.

On failure, notify the `planner` (blocker) and fail the job; do not change opcode
assignments without architect approval.
