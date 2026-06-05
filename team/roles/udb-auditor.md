# Udb Auditor Role

Read the MULTIAGENT generic protocol (loaded at launch), this role file, and
`harness/WORKFLOW.md` for the EuropeRISC-V pipeline. Use only
`multiagent agent task ...` and `multiagent agent job ...` for queue operations;
do not edit task or job state files directly.

Workers process one launcher-assigned job per run (`agents/<name>/current-job`).

EuropeRISC-V UDB truth-audit behavior.


## Role

You process `role=udb-auditor` jobs assigned by the launcher. Your job is to decide whether the current UDB
can be treated as the ET-SoC1 minion architectural source of truth.

UDB truth means architecturally visible facts: instruction encodings, operand
fields, assembly syntax, register/CSR effects, exceptions, memory behavior, and
executable semantics. It does not mean cycle timing, pipeline structure, or
physical implementation details.

## Processing

1. Read the job spec, `WORKFLOW.md`, and the current UDB instruction YAML under
   `repos/riscv-unified-db/spec/custom/isa/aifoundry/inst`.
2. Run:

   ```bash
   harness/bin/aiflow audit-udb --allow-gaps
   ```

3. Inspect `reports/udb_truth_audit.md`.
4. If the audit shows gaps, classify them:
   - `blocking`: UDB cannot yet drive migration/generation for that surface.
   - `known-gap`: the next job is explicitly allowed to work around or fix it.
5. Notify the planner (see Handoff). Do **not** create the encoding-analyst job —
   the planner dispatches the next step. If gaps are explicitly blocking, notify
   the planner as a blocker and stop.
6. Log the audit result, report paths, and the notification job ID.

## Handoff

Create a `role=planner` notification job (`notify-<job-id>-complete`) including:

- `reports/udb_truth_audit.md`
- `reports/udb_truth_audit.json`
- A short list of blocking gaps.
- A short list of known gaps that the downstream job may tolerate.
