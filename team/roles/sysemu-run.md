# Sysemu Run Role

Read the MULTIAGENT generic protocol (loaded at launch), this role file, and
`harness/WORKFLOW.md` for the EuropeRISC-V pipeline. Use only
`multiagent agent task ...` and `multiagent agent job ...` for queue operations;
do not edit task or job state files directly.

Workers process one launcher-assigned job per run (`agents/<name>/current-job`).

Read the MULTIAGENT generic protocol and `WORKFLOW.md`. Run the requested instruction/runtime proof
under `sys_emu` (not decoder edits). For smoke-only Phase 3 jobs, use the smoke
instruction declared by the runtime harness manifest. If the task or job spec
names another instruction, run that instruction's targeted harness instead of
only the smoke.


## Mandatory setup

Run from the **repository root** (MULTIAGENT agents use the repo checkout as cwd;
instance state lives under `~/.multiagent/state/<instance>/`). The Phase 3 wrapper is
`harness/tools/phase3-runtime.sh` (legacy legacy harness tree, kept intact). Point it at the
task runtime harness:

```bash
export AIFLOW_RUNTIME_DIR=harness/runtime/<task-runtime-dir>   # run_sysemu.sh + manifest.yaml
harness/tools/phase3-runtime.sh sysemu
```

The wrapper sources `harness/runtime/env.defaults.sh` and runs
`$AIFLOW_RUNTIME_DIR/run_sysemu.sh`. The harness's `manifest.yaml` declares the
smoke instruction, run script, log path, and PASS marker (see `harness/README.md`
"Runtime / DV").

**If no runtime harness exists for this task** (the clean baseline ships none),
do not fake it and do not run an unrelated smoke as if it proved the target:
create the minimal `harness/runtime/<task>/` (`manifest.yaml`, `run_sysemu.sh`, an
ELF generator that assembles the target instruction) following an existing
harness as the pattern; if you cannot, notify the planner (blocker) and fail the
job. Do **not** call `repos/et-platform/sw-sysemu/build/sys_emu` on the main tree
unless the wrapper fails and you state why.

## Target-aware validation

Read the task/job spec before running. List every requested runtime target
(mnemonic, raw instruction word if available, registers/expected result). The
job log must map each target to a command and a PASS/FAIL line.

## Execute-equivalence (the migration proof)

When the simulator handed an **execute-equivalence vector** (target NEW-encoding
word + reference word + operands), run the **differential**, not just a one-sided
smoke:

1. Run the target program (NEW-encoding word) on the migrated `sys_emu`; capture
   the architectural result (rd / fflags / memory; the **full output vector** for
   packed-SIMD).
2. Run the OLD-word retirement negative on the migrated `sys_emu`: for a
   migration, the OLD word must trap, be rejected, or otherwise not dispatch to
   the migrated instruction unless compatibility/dual-decode is explicit.
3. Run the reference: the same program with the **reference word** — for a
   migration, the instruction at its `old_opcode` on the **baseline** (pre-migration)
   `sys_emu`; for a new instruction, the named equivalent instruction. Capture the
   same state.
4. **Diff the two architectural results — they MUST be identical.** That is the
   proof the re-encoding preserved behavior against the golden (taped-out) emulator.

Record target, OLD-negative, baseline-reference runs, the diff, and PASS/FAIL. If
the simulator provided no vector and only a smoke exists, say so plainly — a
smoke is not an equivalence proof.

## Pass / fail

| Result | Condition |
|--------|-----------|
| **PASS** | The execute-equivalence diff is empty (target ≡ baseline reference) for every requested target, OLD-word retirement in migrated `sys_emu` is proven for migrations, and all run logs are in the job log. Smoke-only is acceptable PASS **only** when the spec explicitly scopes it to a smoke. |
| **FAIL** | Non-zero exit, non-empty diff (target ≠ reference), OLD word still dispatches in migrated `sys_emu`, missing target evidence, or `Error loading ELF` without recovery — notify the planner (blocker); the planner routes a `simulator` fix |

Copy the last 20 lines of the manifest-declared sysemu log into the job log.

## When Done

1. Create `notify-<job-id>-complete` for `planner` with: log path, exit code,
   the PASS/FAIL line, and the runtime-harness dir used. Do **not** create the
   `isa-run-verilator` job — the planner dispatches Phase-3 jobs (it creates
   sysemu-run and verilator-run together).
2. `multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"` only on **PASS**; otherwise `multiagent agent job fail <job-id> --agent-id <your-agent-name> -m "<reason>"` with log excerpt
   (the planner will route the fix).
