# Agentic ISA Migration Demo

**From prompt to proof: one multi-agent run, five public repos, one reproducible
ISA migration snapshot.**

Pinned multi-repo snapshot for a clean-release RISC-V ISA migration carried by
the multi-agent flow.

This repository is intentionally a thin wrapper. It does not copy the component
repositories; it pins them as submodules at the exact commits produced or used by
the agentic flow.

## Clone

```bash
git clone --recurse-submodules https://github.com/AFOliveira/agentic-isa-migration-demo.git
cd agentic-isa-migration-demo
```

If submodules are skipped during clone:

```bash
git submodule update --init --recursive
```

## Contents

| Path | Repository | Pinned SHA |
| ---- | ---------- | ---------- |
| `agentic/multiagent` | `AFOliveira/multiagent` | `a21931dfb358b3be373c3be1f84386f27840afbd` |
| `repos/binutils-gdb` | `AFOliveira/binutils-gdb` | `42f76cd94338df497f1ef5b0ddb029873e63fdad` |
| `repos/riscv-unified-db` | `AFOliveira/riscv-unified-db` | `879e050fbc453d099f53575be294de9602dc3f02` |
| `repos/et-platform` | `AFOliveira/et-platform` | `36ff45f06ebf37df61fc3bea30e277ce696c3206` |
| `repos/core-et` | `AFOliveira/core-et` | `3b95386f29b7c2fd6aa2f8b9ac0bce9e8d8a1a9f` |
| `repos/zephyr` | `AFOliveira/zephyr` | `5ccd6f7231e8ef741ecaa803103498b1344f5554` |

## Scope

The clean-release ISA migration branches align UDB, Binutils, `sys_emu`,
core-et, and Zephyr for the demo flow. Current evidence and limitations are
recorded in `MANIFEST.md`.
