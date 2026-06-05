# Document production release outcomes

## Task

`<task-id>` — set to the active MULTIAGENT task.


## Target

`<implementation-root>`

## Objective

Write durable docs after **final release review passed**. Summarize the E2E
production release: encoding plan, pushed component branches, release manifest,
Phase 3/runtime evidence, final release verification, and how to re-run.

## Sources (read first)

- `harness/WORKFLOW.md` or `harness/WORKFLOW.md`
- task runtime harness README and manifest
- `artifacts/isa-integrate/MANIFEST.md`
- `artifacts/isa-release/MANIFEST.md`
- `$MULTIAGENT_STATE_DIR/jobs/isa-review/log.md` (pre-release review job log under instance state)
- `$MULTIAGENT_STATE_DIR/jobs/isa-release-review/log.md` (final release review job log under instance state)
- Phase 3 logs declared by the task runtime manifest

## Deliverables

Add or update under `implementation/` (pick best existing doc home):

- One page: **EuropeRISC-V production E2E** — pipeline, roles, pushed branches,
  release commits, re-run commands
  (`harness/tools/phase3-runtime.sh`, `aiflow validate`)
- Link to the active encoding plan and summarize each planned instruction's
  MATCH/MASK or recorded blocker status.
- A final audit note showing:
  - `harness/tools/doc-audit <changed-docs>` passed;
  - rerun commands, expected PASS tokens, and log names were compared against the
    release manifest;
  - the document distinguishes implementation/evidence commits from any later
    docs-only commit.

## When Done

- Run `harness/tools/doc-audit <changed-docs>` from the repository root and fix
  any failure before closing the job.
- `multiagent agent job done <job-id> --agent-id <your-agent-name> -m "<summary>"` with list of files changed and the doc-audit PASS line.
- If task id set: suggest `multiagent agent task state <task-id> done` in planner notification
  (`notify-isa-document-complete`).


## MULTIAGENT notes

- Job specs are passed as a file path to `multiagent agent job create`.
- Workers must create a `role=planner` notification job on the same task before `job done` / `job fail` / `job release` (hub-and-spoke dispatch).
- Shared Phase 2 AIFLOW workspace: `harness/jobs/<toolchain-job-id>/workspace/aiflow/` in the repository (not MULTIAGENT instance state). See `harness/docs/aiflow-workspace.md`.
