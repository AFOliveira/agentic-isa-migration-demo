# Phase 2 — RTL decoder

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Generate RTL decoder cases into core-et from the plan's `rtl_ctrl` blocks.

## Workspace

```bash
. jobs/isa-toolchain/workspace/aiflow/env.sh
```

## Processing

`aiflow apply --targets rtl` writes the `32'b…: raw_inst_ctrl = {…}` decode
cases between the `AIFLOW-CUSTOM` markers in
`repos/core-et/hw/ip/minion/frontend/rtl/intpipe_decode.sv`. Each instruction
needs an `rtl_ctrl` block in `harness/config/encoding_plan.yaml` (see the schema
comment there); entries without one are RTL-pending.

For migrations with `old_opcode`, the RTL output must include the NEW row and
retire the OLD row from the migrated decoder unless the plan explicitly declares
compatibility/dual-decode. Record grep/diff evidence for both the NEW word and
the absent/illegal OLD word. Do not make behavior-changing lint fixes outside the
generated decoder row without targeted DV/cosim evidence.

```bash
harness/bin/aiflow apply --force --targets rtl
git -C repos/core-et diff -- hw/ip/minion/frontend/rtl/intpipe_decode.sv
harness/bin/aiflow validate --allow-arch-blockers
```

Run core-et decoder DV if the environment supports Verilator; otherwise record
why skipped:

```bash
make -C repos/core-et/hw/ip/minion/frontend/dv test
make -C repos/core-et lint
```

## When Done

**Sub-chain to dv** (rtl → dv is a direct hand-off, not via the planner): create
`isa-dv` with `-r dv` and spec `.multiagent/specs/isa-dv.md` (same `-t <task-id>`), passing the RTL
worktree/branch, shared `env.sh`, and the exact decode rows added. dv authors a
directed test for your row in your worktree and reports the combined RTL+DV result
to the planner. Do not notify the planner on success. On failure, notify the
planner (blocker) and fail the job.


## MULTIAGENT notes

- Job specs are passed as a file path to `multiagent agent job create`.
- Workers must create a `role=planner` notification job on the same task before `job done` / `job fail` / `job release` (hub-and-spoke dispatch).
- Shared Phase 2 AIFLOW workspace: `harness/jobs/<toolchain-job-id>/workspace/aiflow/` in the repository (not MULTIAGENT instance state). See `harness/docs/aiflow-workspace.md`.
