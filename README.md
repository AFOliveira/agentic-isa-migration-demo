# Agentic ISA Migration Demo

**From prompt to proof: one multi-agent run, five public repos, one reproducible
ISA migration snapshot.**

Pinned multi-repo snapshot for a clean-release RISC-V ISA migration carried by
the multi-agent flow.

This repository is a thin wrapper plus runnable MULTIAGENT configuration. It
does not copy the component repositories; it pins them as submodules at the
exact commits produced or used by the agentic flow.

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
| `.multiagent/` | runnable MULTIAGENT team, roles, and job specs | local config |
| `agentic/multiagent` | `AFOliveira/multiagent` | `7f1dc3c3512cb3388ee742a6fddab4135d37e37d` |
| `repos/binutils-gdb` | `AFOliveira/binutils-gdb` | `42f76cd94338df497f1ef5b0ddb029873e63fdad` |
| `repos/riscv-unified-db` | `AFOliveira/riscv-unified-db` | `879e050fbc453d099f53575be294de9602dc3f02` |
| `repos/et-platform` | `AFOliveira/et-platform` | `36ff45f06ebf37df61fc3bea30e277ce696c3206` |
| `repos/core-et` | `AFOliveira/core-et` | `3b95386f29b7c2fd6aa2f8b9ac0bce9e8d8a1a9f` |
| `repos/zephyr` | `AFOliveira/zephyr` | `5ccd6f7231e8ef741ecaa803103498b1344f5554` |

## Agent Team

The clean-release run used the committed roster in `.multiagent/team.toml`.
The same template is packaged in
`agentic/multiagent/multiagent/templates/teams/isa-migration.toml`.

- `planner-1` and `reviewer-1`: Codex, `gpt-5.5`, `thinking_effort = "xhigh"`.
- Domain workers: Cursor, `composer-2.5`.
- Worker roles: `udb-auditor`, `encoding-analyst`, `compliance`, `toolchain`,
  `rtl`, `dv`, `simulator`, `zephyr`, `integrator`, `sysemu-run`,
  `verilator-run`, `documenter`, `implementer`, and `committer`.
- Per-role prompt markdowns are committed under `.multiagent/roles/`; planner
  job-spec templates are committed under `.multiagent/specs/`.
- The public copy is sanitized: operator-specific hostnames, user directories,
  and repository names are replaced with placeholders.

## MULTIAGENT Use

From a fresh clone with `multiagent` installed:

```bash
multiagent local update
multiagent local team list
multiagent local role list
```

The MULTIAGENT configuration is runnable outside the original operator
environment, but the full ISA migration workload still needs an implementation
checkout that provides the referenced harness scripts, tools, board/runtime
setup, and operator-specific remote workload host.

## Scope

The clean-release ISA migration branches align UDB, Binutils, `sys_emu`,
core-et, and Zephyr for the demo flow. Current evidence and limitations are
recorded in `MANIFEST.md`.
