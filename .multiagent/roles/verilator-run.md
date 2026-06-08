# Verilator Run Role

Read the MULTIAGENT generic protocol (loaded at launch), this role file, and
`harness/WORKFLOW.md` for the EuropeRISC-V pipeline. Use only
`multiagent agent task ...` and `multiagent agent job ...` for queue operations;
do not edit task or job state files directly.

Workers process one launcher-assigned job per run (`agents/<name>/current-job`).

Read the MULTIAGENT generic protocol and `WORKFLOW.md`. Validate RTL decode behavior with **core-et's
own Verilator DV** (`repos/core-et`), driven by `harness/tools/phase3-runtime.sh verilator` from the repository root. Local Verilator is configured by `harness/runtime/env.defaults.sh`.
Follow core-et's DV rules in `repos/core-et/AGENTS.md` (DV section) and
`repos/core-et/dv/rtlcosim/README.md`.

If the task or job spec names any instruction, opcode, RTL decoder change, or
runtime behavior, validate that target specifically. Do not treat a neighboring
or pre-existing test pass as evidence for a different target.


## Mandatory setup

```bash
harness/tools/phase3-runtime.sh verilator   # from repository root
```

The wrapper sources `harness/runtime/env.defaults.sh` (Verilator on PATH) and runs
core-et's native make DV:

```bash
make -C repos/core-et/hw/ip/minion/frontend/dv test   # decoder block DV (intpipe_decode_*)
make -C repos/core-et lint                             # zero warnings
make -C repos/core-et/dv/rtlcosim test                # cycle-by-cycle cosim (needs ORIG_ROOT)
```

cosim needs `ORIG_ROOT` pointing at the original CORE-ET RTL tree. For a
production migration task, missing `ORIG_ROOT` is a blocker; find or create the
baseline worktree before marking the job complete. Only dry-run tasks may record
missing cosim as a residual.

## Target-aware validation

1. Read the task and job specs and list the exact RTL targets to prove
   (mnemonic, raw instruction word, and expected decode/behavior).
2. Confirm the core-et decoder block DV includes the **directed test** authored by
   the `dv` role for the target (a case driving the exact instruction word and
   asserting its decoded `raw_inst_ctrl`), and that it appears in the passing run.
   You are the runner, not the test author — if the directed test is missing or
   does not exercise the target, notify the planner for `isa-dv` fix routing (do
   not write the test yourself; author ≠ runner ≠ gate).
3. For every migrated instruction with `old_opcode`, confirm the directed test
   also proves OLD-word retirement in the migrated decoder (illegal or non-target)
   unless compatibility/dual-decode is explicit. A test that accepts the OLD row
   in the migrated decoder is a failure.
4. **RTL equivalence (when `ORIG_ROOT` is available).** For a migration, the
   strongest RTL proof is `dv/rtlcosim` comparing the migrated module against the
   **original** CORE-ET module cycle-by-cycle on identical stimulus
   (`make -C repos/core-et/dv/rtlcosim test` with `ORIG_ROOT` set). For the decoder,
   that means the migrated decode produces the same control as the original on the
   target's stimulus — the RTL analogue of the sysemu execute-equivalence diff. If
   `ORIG_ROOT` is unset, fail/block in production scope rather than implying full
   equivalence.
5. The job log must include a coverage matrix mapping each required target to
   the make command, log path, and `TEST PASSED`/FAIL line — including whether the
   rtlcosim equivalence ran or was skipped (ORIG_ROOT absent).

## Architectural writeback targets

When the task asks for an architecturally visible result, register-file state, a
retired instruction, or named destination-register value, **do not stop at a
decoder, toy wrapper, or isolated execute-unit proof**. Those checks are useful
supporting evidence, but they are not a Verilator proof of architectural
writeback.

For architectural-state requests, first attempt a full-core or pipeline-level
Verilator harness that instantiates enough of the real RTL path to cover:

Architectural-state coverage template:

| Required surface | Required direct evidence |
|------------------|--------------------------|
| Instruction fetch or injection | exact raw instruction word reaches real decode |
| Register operands | real register-file/read path or documented pipeline interface supplies the requested source operands |
| Decode controls | expected decode row selects the requested execute function, operands, width, and destination register |
| Execute path | real RTL execute path produces the requested value |
| Writeback/retirement | real writeback/register-file state, commit bus, or equivalent architectural observation shows the requested destination register value |

Acceptable proof examples include:

- a full core/minion Verilator simulation that loads a tiny program and observes
  the requested destination register value after retirement;
- a pipeline/register-file harness that drives the real instruction through
  decode/execute/writeback and observes the actual write port or register file;
- an existing ET diagnostic/minion harness adapted to print a PASS line naming
  the instruction, raw word, source operands, and destination result.

If a full-core harness cannot be built in the current environment, log the exact
blocker with paths and commands tried: missing top module, missing memory model,
reset/clock/boot protocol, inaccessible generated files, unsupported Verilator
constructs, or absent architectural observation point. Then notify the planner
with the defect, suggested owning role, and evidence instead of marking
architectural writeback as PASS.

Only call a module-level execute proof PASS for the narrower claim "real RTL
execute arithmetic." If the task asks for architectural register state, the
coverage matrix must show either direct full-core/pipeline writeback evidence or
an explicit blocker.

## PASS label discipline

Do not name a module-level harness `ARCH_WB_PASS`, `architectural writeback`, or
`retirement` unless it instantiates the production writeback/commit path being
claimed. If the harness instantiates decode/RF/ALU but uses a testbench FSM,
testbench commit signal, forced signal, or C++-driven writeback, label it as a
focused module-level observation, for example `MODULE_WB_OBS_PASS`, and state
the exact unsupported architectural claim in the coverage matrix.

If an existing wrapper or test prints an over-broad PASS label, update the label
or reject the job before completion. Reviewers are instructed to treat broad
labels as suspect until the harness source proves the boundary.

## Pass / fail

| Result | Condition |
|--------|-----------|
| **PASS** | core-et decoder block DV prints `TEST PASSED` for the required target, migration OLD-word retirement is proven in the migrated decoder, `make -C repos/core-et lint` is clean, and the required migration cosim passes with `ORIG_ROOT` set. If the task requires architectural state/register writeback, PASS requires full-core or pipeline-level evidence observing that state, not only isolated decode or ALU-module evidence. |
| **FAIL** | Any required target is missing from DV, OLD word remains live in migrated decode, `make` exits non-zero, or lint/cosim fails — notify the planner with the defect, evidence, and suggested owning role (`rtl` or `verilator-run`). |

**Do not** mark the job done on lint alone. If `make` reports
`verilator: command not found`, fix `harness/runtime/env.defaults.sh` and re-run.

Optional notes in the log (not required for PASS):
- `make -C repos/core-et coverage-report` (per-IP coverage)

## When Done

1. `notify-isa-run-verilator-complete` for `planner` with: Verilator path used,
   sim PASS/FAIL, log path.
2. `multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"` only if PASS criteria above met.
