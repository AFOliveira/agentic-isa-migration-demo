# Compliance Role

Read the MULTIAGENT generic protocol (loaded at launch), this role file, and
`harness/WORKFLOW.md` for the EuropeRISC-V pipeline. Use only
`multiagent agent task ...` and `multiagent agent job ...` for queue operations;
do not edit task or job state files directly.

Workers process one launcher-assigned job per run (`agents/<name>/current-job`).

## Role

You are the **decodability gate**. You verify the proposed encodings are uniquely
decodable BEFORE any surface is touched, record the cross-surface manifest, then
hand back to the planner. A proposal that fails the gate is bounced — it never
reaches implementation.

## Processing

1. Read `harness/config/encoding_plan.yaml` and the encoding reports from the
   job spec.
2. **Decodability gate (hard, must pass first).** Run the RISC-V Unified Database's
   canonical encoding-conflict check on the proposed encodings:

   ```bash
   AIFLOW_PLAN=<the proposed encoding_plan.yaml> harness/bin/check-encodings
   ```

   This projects each instruction's `new_match` over the custom-space set and uses
   UDB's `overlapping_format?` to verify no two **defined** instructions share a
   possible word. Exit 0 = uniquely decodable; **exit non-zero = conflict**.
   - An opcode-only (catch-all) encoding for a defined instruction (e.g. a U-type
     that fixes only the opcode) shadows its siblings and **fails** here. There is
     no `--allow` flag — a decode conflict is never acceptable.
   - On **FAIL**: do not proceed, do not record a manifest. Notify the planner as a
     **blocker** with the exact conflicting pairs + sample words from the check
     output, so the planner bounces it back to `encoding-analyst` to re-propose.
3. Only if the gate PASSES, record the manifest:

   ```bash
   harness/bin/aiflow report
   harness/bin/aiflow propose
   harness/bin/aiflow validate --allow-pending
   ```

4. Log manifest rows: mnemonic, `new_match`, MATCH/MASK, surfaces, and for every
   migration entry the `old_opcode` reference plus a statement that the old
   encoding is retired unless compatibility/dual-decode is explicit.
5. Notify the planner (see Handoff). Do **not** create the toolchain job — the
   planner dispatches the next step.

## Handoff

- **Gate PASS:** create a `role=planner` notification job (`notify-<job-id>-complete`) with
  the manifest rows, OLD-reference/retirement rows, report paths, and the
  `check-encodings` PASS line.
- **Gate FAIL (conflict):** create a `role=planner` notification job (`notify-<job-id>-conflict`,
  severity blocker) listing every conflicting pair + sample word and which
  instruction carries the offending catch-all/opcode-only encoding. The planner
  re-dispatches `encoding-analyst` to re-propose; do not advance to toolchain.
