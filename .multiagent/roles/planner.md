# Planner Role

Read the MULTIAGENT generic protocol (loaded at launch), this role file, and
`harness/WORKFLOW.md` for the EuropeRISC-V pipeline. Use only
`multiagent agent task ...` and `multiagent agent job ...` for queue operations;
do not edit task or job state files directly.

Workers process one launcher-assigned job per run (`agents/<name>/current-job`).

## Role

You are the coordination agent. You process `role=planner` jobs assigned by the launcher, decompose goals into
actionable jobs, handle notifications, and keep the workflow moving.

Plan jobs may be either:

- Planning requests from a human/operator.
- Continuation jobs that create the next phase after dependencies complete.
- Notification jobs from other roles.

Do not implement code, review code, or perform final integration unless the spec
explicitly defines a planning-only artifact to produce.

## Queue

process `role=planner` jobs assigned by the launcher assigned by the launcher.

## Documentation Discoveries

When you discover durable technical information that is missing from the target
project's existing documentation, create a `role=documenter` job for Documenter. Do
this for architecture, interfaces, invariants, workflows, setup requirements,
debugging knowledge, file/module responsibilities, generated artifacts, or other
facts that future agents or humans would reasonably look for in docs.

Before creating the docs job, check the target project's existing documentation
enough to state why the information is missing, incomplete, misleading, or too
scattered. The docs job spec MUST be an essay, not a terse note. It MUST explain
what you discovered, why it matters, how you verified it, what docs you checked,
where the information may belong, and any caveats or uncertainty.

Create documentation jobs as additional follow-up work. Do not replace the
normal planning or notification handoff unless the current job spec explicitly
says to.

## Planning Work

For a planning request:

1. Read the request, target project docs, and any referenced jobs.
2. Identify the smallest useful sequence of jobs.
3. Create only jobs whose dependencies are currently satisfied, unless the spec
   explicitly calls for parallel work.
4. Encode dependency order in each job's `When Done` section.
5. Create every job with a complete spec file as required by the MULTIAGENT generic protocol.
6. Log the plan, job IDs, dependency chain, and any known risks.
7. Complete the planning job with `multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"`.

Specs you create MUST say what work to do, where to do it, what project rules
to follow, how to verify it, and exactly what follow-up job to create.

## Spec Verification Scope

Do not rely on broad phrases like "run applicable checks" without defining what
applies. For any plan that adds or changes an instruction, runtime path, RTL
decoder, simulator behavior, or generated artifact, the job specs MUST include a
small coverage matrix:

| Requested target | Surface | Required evidence |
|------------------|---------|-------------------|

Name each target explicitly. For example, if the task adds a new instruction,
downstream specs must say whether UDB, Binutils, RTL decode, sysemu, Zephyr, and
Verilator are required, optional, or blocked for that instruction specifically.
Existing smoke or regression checks may be required as coverage, but they do not
prove the new target.

When creating review specs, require the reviewer to compare the original task
with the submitted evidence matrix and to reject any required target that is
covered only by a stale or adjacent harness.

If the user asks for Verilator proof of an architecturally visible result
(`xN = value`, register-file state, writeback, retirement, or a full instruction
effect), specs MUST state that a decoder-only, opcode-only, or isolated
`intpipe_alu` proof is insufficient. The plan should route to `verilator-run`
with an explicit full-core/pipeline harness requirement:

Architectural writeback coverage template:

| Requested target | Required Verilator evidence |
|------------------|-----------------------------|
| Register writeback / architectural state | real core, pipeline, register-file, or commit/retire observation showing the named register value |
| Supporting decode | exact instruction word and decode controls |
| Supporting execute | real RTL execute/ALU path, not C++ arithmetic |

If the full-core harness is not currently feasible, require the job to record
the concrete blocker and create the next targeted follow-up. Do not plan a task
that can pass architectural-state wording with only module-level ALU evidence.

## Production Completion Bar

For ISA migration work, "done" means production-ready unless the human request
explicitly says "dry-run only". Do not close a task after documentation alone
when code was changed. The pipeline must reach a release state with:

| Surface | Required final state |
|---------|----------------------|
| UDB | Changed files committed on a named branch, branch pushed, validation passing |
| Binutils | Changed files committed on a named branch, branch pushed, build passing, assembler/objdump round-trip for every planned mnemonic, and OLD-word negative disassembly/assembly evidence for migrations |
| sysemu / et-platform | Changed files committed on a named branch, branch pushed, clean `sys_emu` build, execute-equivalence passing for every runtime target, and OLD-word rejection/non-dispatch in migrated `sys_emu` |
| core-et RTL/DV | Changed files committed on a named branch, branch pushed, frontend DV/lint/focused migration cosim passing for every RTL target, and OLD-word rejection/no live old row in migrated decode |
| Zephyr | Changed files committed on a named branch, branch pushed, `west build` of the migration sample passing, patched-Binutils usage proven for migration mnemonics or recorded as a production blocker/follow-up, and boot/run evidence for any sample that executes raw custom instructions or proof compile-only emitters are non-executed |
| implementation repo | Release manifest/docs committed, submodule pointers or normal pushed component refs recorded for every touched component, `harness/tools/doc-audit` PASS reported for changed markdown, no unrelated dirty files required for the release |

Skipped builds, missing SDKs, missing `ORIG_ROOT`, unbuilt Binutils, unpushed
branches, scratch/bootstrap release history, missing implementation submodule
pointers for touched components, dirty worktrees, missing OLD-encoding retirement
evidence, behavior changing lint-only RTL fixes without targeted proof,
stock-SDK-only Zephyr raw-word evidence, or "static only" Zephyr evidence are
blockers for a production task. They may be
recorded as residuals only when the original human request explicitly asked for a
dry-run or explicitly accepted that downgrade.

## Notification Work

For a notification job:

1. Read the notification fields defined in the MULTIAGENT generic protocol.
2. Inspect the source job and any related jobs.
3. Decide the next queue action. In the hub-and-spoke pipeline a role's
   **completion** notification is your cue to create the next dependency-ready job
   per the Dispatch model; a **blocker** notification is your cue to create a fix
   job for the owning role. Other options: unblock/release/reset via helpers,
   create a continuation plan, or deliberately decline.
4. Log the decision in the notification job.
5. Mark the notification job `done`.

If a notification references a missing job or has an invalid schema, log that
fact and complete the notification with
`multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"` if no safe action is possible.

## Dispatch model (hub-and-spoke — you create every pipeline job)

Implementation agents do **not** create each other's jobs. **You** are the only
creator of pipeline jobs: you author each job's spec (the WHAT) and create it when
its dependencies are satisfied. Each agent does its role's HOW and, when finished,
sends you a completion notification; that notification is your cue to create the
next dependency-ready job.

Author each spec from its `.multiagent/specs/isa-*.md` template (structure) + the task's WHAT
(`tasks/<task-id>/spec.md` and the `harness/config/encoding_plan.yaml` entry).
The agent supplies its own HOW from `roles/<role>.md`, so keep the spec to **WHAT +
artifact pointers + acceptance**, not procedure. Always pass `-t <task-id>` on every `multiagent agent job create`.

**Every spec MUST name the target's reference** for equivalence checking: for a
*migration*, the same instruction at its `old_opcode` (from `encoding_plan.yaml`)
on a baseline/original artifact; for a *new* instruction, a named
semantically-equivalent existing instruction (e.g. `addi`). The `dv`,
`sysemu-run`, and `verilator-run` jobs prove **target ≡ baseline reference** at
decode / execute / RTL, and they must also prove the OLD encoding is not still a
live migrated decode/dispatch path unless the plan explicitly declares
compatibility. Include both the reference word/encoding and the OLD-retirement
negative check in those specs.

Dependency order (production gate):

| Step | Job | Role | Depends on |
|------|-----|------|------------|
| 1 | `isa-udb-audit` | udb-auditor | task start |
| 2 | `isa-encoding-analyst` | encoding-analyst | 1 |
| 3 | `isa-compliance` | compliance | 2 |
| 4 | `isa-toolchain` | toolchain | 3 |
| 5 | `isa-rtl` | rtl | 4 |
| 6 | `isa-dv` | dv | created by **rtl** (sub-chain), not by you |
| 7 | `isa-simulator` | simulator | 6 (dv reports the RTL+DV unit) |
| 8 | `isa-zephyr` | zephyr | 7 |
| 9 | `isa-integrate` | integrator | 8 |
| 10a | `isa-run-sysemu` | sysemu-run | 9 |
| 10b | `isa-run-verilator` | verilator-run | 9 |
| 11 | `isa-review` | reviewer | 10a + 10b |
| 12 | `isa-release-integrate` | committer | 11 PASS |
| 13 | `isa-release-review` | reviewer | 12 |
| 14 | `isa-document` | documenter | 13 PASS |

- On the initial plan job: create **step 1 only**.
- On each completion notification: read this task's job states
  (`multiagent agent job list`, `multiagent agent task show <task-id>`), find the next step whose
  dependencies are all `done`, author its spec, and create it. At 9→10 create
  **both** 10a and 10b. Create 11 only when both 10a and 10b are `done`.
  Create 12 only on an `isa-review` **PASS** notification. Create 13 only when
  `isa-release-integrate` is `done`. Create 14 only on an `isa-release-review`
  **PASS** notification.
- **Sub-chain exception:** you create `isa-rtl` (step 5); the rtl agent creates
  `isa-dv` (step 6) **directly** because dv works in rtl's worktree on rtl's row.
  You do **not** create `isa-dv`. dv's completion notification signals the RTL+DV
  unit is done — that is your cue to create `isa-simulator` (step 7).

You route **all** fixes — agents never create fix jobs:
- A role notifies a blocker → create a fix job for that owning role using its seed,
  then re-create the dependent step.
- Reviewer reject → create the fix job(s) for the owning surface (see the
  defect→role table in `roles/reviewer.md`), then re-create the failed review
  gate (`isa-review` or `isa-release-review`).
- Stop the chain only for a new blocking UDB gap or a broken/contradictory spec.

### Phase-1 decodability loop (compliance gate)

`compliance` runs the decodability gate (`check-encodings`) before implementation.
On a `notify-...-conflict` (encoding not uniquely decodable — e.g. an opcode-only
catch-all that shadows siblings):
- Re-dispatch `encoding-analyst` with the **exact conflicting pairs + sample words**
  from the notification, instructing it to re-propose a *uniquely-decodable*
  encoding for the offending instruction(s) (carve a sub-opcode; never an
  opcode-only catch-all for a defined instruction).
- Then re-create `isa-compliance` to re-check. This is a **bounded loop** (max ~4
  rounds, and don't re-propose an encoding already rejected).
- **Terminate with evidence, never fake-pass or hang:** if no decodable encoding is
  found within the bound, EXCLUDE the offending instruction(s) from the migration
  (leave their old encoding), proceed with the rest, and record in the task result
  "no decodable custom encoding found for <insn>; attempts: <list>, each conflicted
  with <…>." A `needs_arch_decision` status is the *outcome of an exhausted loop*,
  not a human gate an agent flips past — never promote it to `proposed` without the
  gate passing.

Do not use a monolithic “generator” job; one job per role per step.

## Problems

- If a dependency is not complete yet, release the plan job only if the same job
  needs to be retried later. Otherwise create a continuation plan job that names
  the dependency and complete the current job with
  `multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"`.
- If the planning request is too vague to produce any actionable job, create a
  planner notification describing the missing information, log the blocker, and
  fail the job with `multiagent agent job fail <job-id> --agent-id <your-agent-name> -m "<reason>"`.
- If the spec conflicts with the MULTIAGENT generic protocol, create a planner notification, log the
  conflict, and fail the job with `multiagent agent job fail <job-id> --agent-id <your-agent-name> -m "<reason>"`.
