# Step 11 — pre-release review

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Holistic quality gate after **all** propagation and Phase 3 runtime. Decide
whether the migration is ready for production release integration, or send a
structured rework list to the **planner** to re-queue fixes.

## Prerequisites

- Phase 2 jobs done: toolchain, rtl, simulator, zephyr
- `isa-integrate` **done** (`artifacts/isa-integrate/`)
- `isa-run-sysemu` **done**
- `isa-run-verilator` **done**

## Workspace

```bash
# From repository root (MULTIAGENT workers):
. harness/jobs/<toolchain-job>/workspace/aiflow/env.sh
# From harness/:
# . jobs/<toolchain-job>/workspace/aiflow/env.sh
```

Inspect `branches.md`, worktree diffs, `artifacts/isa-integrate/MANIFEST.md`, and
Phase 3 logs.

## Processing

1. Static validation:

   ```bash
   cd <implementation-root>
   . harness/jobs/<toolchain-job-id>/workspace/aiflow/env.sh
   harness/bin/aiflow validate --allow-arch-blockers
   ```

2. Phase 3 (re-run if job logs are weak):

   ```bash
   harness/tools/phase3-runtime.sh both   # from repository root; wrapper under harness/tools/
   ```

3. Check every requested instruction target across both surfaces (sysemu in
   `et-platform`, RTL decode in core-et).
4. For every migrated instruction with `old_opcode`, verify the ISA Migration
   Release Contract from `harness/WORKFLOW.md`: NEW encoding accepted, baseline
   reference equivalence proven, and OLD encoding retired/rejected in patched
   Binutils, migrated core-et RTL/DV, and migrated `sys_emu`.

5. For any Verilator claim about architectural state, register writeback,
   retirement, or a destination-register value, inspect the harness source and
   add the reviewer proof-boundary matrix from `roles/reviewer.md`. Do not accept
   a PASS label as evidence unless the instantiated RTL boundary matches the
   claim.

6. Logs:
   - sysemu log declared by the task runtime manifest
   - core-et decoder DV output (`hw/ip/minion/frontend/dv test`) with `TEST PASSED`
   - `make -C repos/core-et lint` clean

## Pass

- `aiflow validate` exit 0 for planned entries
- Cross-surface agreement; Phase 3 PASS lines present
- Integration bundle matches reviewed worktrees
- Patched Binutils build + round-trip evidence exists for every planned mnemonic
- OLD-word retirement evidence exists for every migrated instruction on every
  touched surface, unless compatibility/dual-decode is explicit in the plan
- Zephyr `west build` evidence exists for the migration sample, and the review
  states whether the artifact used the patched Binutils or only the stock Zephyr
  SDK. Stock-SDK raw-word compile-only evidence is not enough for production
  unless it is recorded as a blocker/follow-up.
- Zephyr boot/run evidence exists on `sys_emu` for any runtime sample path, and
  full-system Verilator boot/run is present when a target exists; otherwise the
  missing target/path is recorded as an infrastructure gap.
- Compile-only Zephyr emitters have non-execution proof.
- No required production evidence is downgraded to a residual
- Architectural-state/register-writeback claims are backed by full-core,
  pipeline-level, or real register-file proof. Module-level observations are
  recorded as narrower evidence, not accepted as the broader claim.

## When Done — notify the planner with your verdict (you create no follow-up job)

Hub-and-spoke: you decide pass / changes-needed / blocked and notify the planner;
the planner dispatches the follow-up (see `roles/reviewer.md` + `roles/planner.md`).

- **Pass:** create `notify-<job-id>-pass` for `planner` with the coverage matrix.
  The planner creates `isa-release-integrate`.
- **Changes needed:** create `notify-<job-id>-changes` for `planner` with a table
  **surface | symptom | owning role | paths/commands** and whether to re-run
  Phase 3 only or repeat Phase 2. The planner creates the fix job(s) and re-creates
  `isa-review`. Do not create fix jobs or `isa-document` yourself.
- **Blocked:** `multiagent agent job fail <job-id> --agent-id <your-agent-name> -m "<reason>"` + planner notification if worktrees/logs are missing.

Then run `multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"` on the review job.


## MULTIAGENT notes

- Job specs are passed as a file path to `multiagent agent job create`.
- Workers must create a `role=planner` notification job on the same task before `job done` / `job fail` / `job release` (hub-and-spoke dispatch).
- Shared Phase 2 AIFLOW workspace: `harness/jobs/<toolchain-job-id>/workspace/aiflow/` in the repository (not MULTIAGENT instance state). See `harness/docs/aiflow-workspace.md`.
