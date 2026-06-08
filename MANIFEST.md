# Agentic ISA Migration Demo Manifest

Created: 2026-06-05
Updated: 2026-06-08 (public evidence and Zephyr runtime status)

## Agentic System

| Component | SHA | Notes |
| --------- | --- | ----- |
| `AFOliveira/multiagent` | `7f1dc3c3512cb3388ee742a6fddab4135d37e37d` | Multi-engine worker adapters, resumable sessions, sanitized workload-placement policy, and the packaged ISA migration team template. Verified: team TOML parses; `tests/test_packaging.py` passed. |

## Agent Team

The public demo commits the clean-release roster at `.multiagent/team.toml`,
per-role prompts at `.multiagent/roles/*.md`, and planner job-spec templates at
`.multiagent/specs/*.md`. It contains 16 workers:

- Codex `gpt-5.5` with `thinking_effort = "xhigh"`: `planner-1`,
  `reviewer-1`.
- Cursor `composer-2.5`: `udb-auditor-1`, `encoding-analyst-1`,
  `compliance-1`, `toolchain-1`, `rtl-1`, `dv-1`, `simulator-1`, `zephyr-1`,
  `integrator-1`, `sysemu-run-1`, `verilator-run-1`, `documenter-1`,
  `implementer-1`, and `committer-1`.

The role/spec markdowns are sanitized from the operator environment:
operator-specific hostnames, user directories, and repository names are
represented with placeholders.

## Runnable layout

This top-level repository is **not standalone-runnable** for the full ISA
migration pipeline. It commits `.multiagent/` and submodule pins only. The
AIFLOW harness, workflow contract, runtime scripts, encoding plan, and job
workspace paths referenced by the specs live in a separate implementation
checkout (see `README.md` and `EVIDENCE.md`). No equivalent `harness/` tree is
committed here.

## ISA Migration Stack

| Component | SHA | Visibility | Role |
| --------- | --- | ---------- | ---- |
| `AFOliveira/binutils-gdb` | `42f76cd94338df497f1ef5b0ddb029873e63fdad` | Public | Patched XAIFET opcode tables for the clean-release migration targets. |
| `AFOliveira/riscv-unified-db` | `879e050fbc453d099f53575be294de9602dc3f02` | Public | UDB encodings for the clean-release migration targets. |
| `AFOliveira/et-platform` | `36ff45f06ebf37df61fc3bea30e277ce696c3206` | Public | `sys_emu` migration dispatch, OLD-encoding retirement, `erbium_emu` build for Zephyr runtime. |
| `AFOliveira/core-et` | `3b95386f29b7c2fd6aa2f8b9ac0bce9e8d8a1a9f` | Public | RTL/DV/Verilator gate for `packb`, `bitmixb`, and `aif.europeriscvsummit`. |
| `AFOliveira/zephyr` | `5ccd6f7231e8ef741ecaa803103498b1344f5554` | Public | Patched-Binutils Zephyr ISA migration smoke proof, non-execution contract, and `erbium_emu` runtime notes. |

## Validation Snapshot

Source experiment: `multiagent/clean-release-flow-20260604` at implementation
commit `97a746dcc8f7f8f8e7f3fb5f2cdb31fa42e98020` (summarized here; harness and
job logs not committed in this demo repo).

The clean-release evidence recorded by the source run showed:

- AIFLOW validation passed for 9 proposed targets; `fbci.ps` and `fbci.pi`
  remained excluded pending architecture decisions.
- Binutils round-trip plus OLD-encoding retirement passed **9/9** and **8/8**
  respectively.
- `sys_emu` strict target-to-baseline equivalence passed **`SYSEMU_STRICT_EQUIV_PASS 9/9`**.
- core-et frontend DV (**14/14**), repo lint (**65/0**), and migration rtlcosim
  (**COSIM PASSED** with baseline `ORIG_ROOT`) passed for RTL-scoped targets.
- Zephyr patched `as-new` plus `objdump` mnemonic verification passed **9/9**.
- Zephyr `west build -p always -b erbium_minion
  samples/aifoundry/isa_migration_smoke` passed.
- Zephyr non-execution proof showed `main()` does not call the custom-instruction
  emitters in the runtime path exercised.
- **`erbium_emu` runtime:** boot of the built `zephyr.elf` with
  `-reset_pc 0x40000200` observed Zephyr OS boot and the sample printk line
  (`isa-migration-smoke: strict compile-only PASS ...`). See `EVIDENCE.md`.

Detailed pass tokens, command shapes, and proof boundaries are in **`EVIDENCE.md`**.

## Known Limitations

- **Harness not included:** rerunning the MULTIAGENT pipeline requires an
  external implementation checkout with AIFLOW/workflow tooling (not pinned in
  this demo).
- **Zephyr runtime boundary:** patched build + non-execution + **`erbium_emu`**
  boot/run printk evidence passed on the pinned et-platform/Zephyr SHAs. Direct
  ET-SoC1 **`sys_emu`** with shires/reset settings used for other boards is the
  **wrong emulator path** for `erbium_minion` and must not be treated as the
  headline runtime blocker.
- **Full-system Verilator Zephyr boot** remains unavailable: no Verilator SoC
  model + ELF loader + UART capture + Zephyr runner integration for
  `isa_migration_smoke` in the pinned component trees. Decoder-level DV/rtlcosim
  is valid decode evidence only, not OS runtime proof.
- **Publication:** this snapshot summarizes evidence from the source experiment;
  final public release integration/push of all surfaces may still be in progress.
  Verify fork branch heads against the SHAs above before treating the demo as a
  released tag.
