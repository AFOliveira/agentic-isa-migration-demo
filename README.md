# Agentic ISA Migration Demo

**From prompt to proof: one multi-agent run, five public component repos, and a
documented ISA migration snapshot.**

Pinned multi-repo snapshot for a clean-release RISC-V ISA migration carried by
the multi-agent flow.

This repository is a **reference snapshot**: runnable MULTIAGENT team
configuration plus exact submodule pins. It does **not** include the full
implementation harness (AIFLOW scripts, workflow contract, runtime vectors, or
job log artifacts) that the agents used during the source experiment.

## Runnable layout (read this first)

| In this snapshot | Not in this snapshot |
| ---------------- | -------------------- |
| `.multiagent/` team, roles, job-spec templates | `harness/` tree (`WORKFLOW.md`, `bin/aiflow`, `tools/phase3-runtime.sh`, `config/encoding_plan.yaml`, `jobs/.../workspace/aiflow/`, integration bundles) |
| Public submodule pins under `repos/*` | Operator MULTIAGENT instance state or private job logs |
| `MANIFEST.md`, `EVIDENCE.md` (summary proof boundaries) | Raw command logs, patches, or a self-contained re-run script |

**You cannot rerun the full ISA migration workload from this top-level tree
alone.** The committed `.multiagent/specs/` and `.multiagent/roles/` files
reference paths such as `harness/WORKFLOW.md`, `harness/tools/phase3-runtime.sh`,
and `harness/jobs/isa-toolchain/workspace/aiflow/` that exist only in a separate
**implementation checkout** used during the source experiment (branch
`multiagent/clean-release-flow-20260604` at commit
`97a746dcc8f7f8f8e7f3fb5f2cdb31fa42e98020` — summarized in `EVIDENCE.md`, not
published as part of this demo repo).

What you **can** do from this snapshot:

- Inspect the sanitized agent team and job-spec templates (`.multiagent/`).
- Clone and build the pinned public component forks at the exact SHAs below.
- Read `MANIFEST.md` and `EVIDENCE.md` for pass tokens and proof boundaries.
- Follow runtime/build notes inside the pinned Zephyr fork
  (`samples/aifoundry/isa_migration_smoke/README.md`).

What requires the external implementation checkout (or equivalent tooling you
supply yourself):

- Running `aiflow`, decodability gates, Phase 2 workspace preparation, Phase 3
  strict sysemu/Verilator harness scripts, and MULTIAGENT job replay end-to-end.

No public-safe harness source is pinned in this repository today. Prefer treating
this demo as **inspectable reference material**, not a standalone runnable
pipeline, until a harness publication path exists.

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
| `EVIDENCE.md` | — | Sanitized proof-boundary appendix for the pinned SHAs |

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

The MULTIAGENT **configuration** is runnable for inspection and team listing, but
pipeline jobs in `.multiagent/specs/` assume the missing implementation harness
described above.

## Scope

The clean-release ISA migration branches align UDB, Binutils, `sys_emu`,
core-et, and Zephyr for the demo flow. Current evidence summaries and
limitations are in `MANIFEST.md`; inspectable pass tokens and proof boundaries
are in `EVIDENCE.md`.
