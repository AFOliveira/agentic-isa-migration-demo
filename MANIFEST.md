# AGENTIC DEMO Manifest

Created: 2026-06-05

## Agentic System

| Component | SHA | Notes |
| --------- | --- | ----- |
| `AFOliveira/multiagent` | `a21931dfb358b3be373c3be1f84386f27840afbd` | Multi-engine worker adapters and resumable sessions. Local tests: `82 passed`. |

## ISA Migration Stack

| Component | SHA | Visibility | Role |
| --------- | --- | ---------- | ---- |
| `AFOliveira/binutils-gdb` | `42f76cd94338df497f1ef5b0ddb029873e63fdad` | Public | Patched XAIFET opcode tables for the clean-release migration targets. |
| `AFOliveira/riscv-unified-db` | `879e050fbc453d099f53575be294de9602dc3f02` | Public | UDB encodings for the clean-release migration targets. |
| `AFOliveira/et-platform` | `36ff45f06ebf37df61fc3bea30e277ce696c3206` | Public | `sys_emu` migration dispatch, OLD-encoding retirement, and runtime fixes. |
| `AFOliveira/core-et` | `3b95386f29b7c2fd6aa2f8b9ac0bce9e8d8a1a9f` | Public | RTL/DV/Verilator gate for `packb`, `bitmixb`, and `aif.europeriscvsummit`. |
| `AFOliveira/zephyr` | `5ccd6f7231e8ef741ecaa803103498b1344f5554` | Public | Patched-Binutils Zephyr ISA migration smoke proof and runtime notes. |

## Validation Snapshot

Source experiment: `multiagent/clean-release-flow-20260604` at implementation
commit `97a746dcc8f7f8f8e7f3fb5f2cdb31fa42e98020`.

The clean-release evidence recorded by the server run showed:

- AIFLOW validation passed for 9 proposed targets; `fbci.ps` and `fbci.pi`
  remained excluded pending architecture decisions.
- Binutils round-trip plus OLD-encoding retirement passed 9/9.
- `sys_emu` strict equivalence passed 9/9.
- core-et frontend DV, lint, repo lint, and RTL cosim evidence passed.
- Zephyr patched `as-new` plus `objdump` mnemonic verification passed 9/9.
- Zephyr `west build -p always -b erbium_minion
  samples/aifoundry/isa_migration_smoke` passed.
- Zephyr non-execution proof showed `main()` does not call the custom-instruction
  emitters in the runtime path.

## Known Limitations

- Zephyr full runtime remains an infrastructure gap: `sys_emu` boot hit trap
  recursion at PC `0x0`, and no full-system Verilator Zephyr boot target was
  present in the tree.
- The pinned component commits are published on public
  `multiagent/isa-migration-clean-release-20260604/isa-migration-release`
  branches in the `AFOliveira` forks.
