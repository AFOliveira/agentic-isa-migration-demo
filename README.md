# AGENTIC DEMO

Pinned multi-repo snapshot for the agentic ISA migration demo.

This repository is intentionally a thin wrapper. It does not copy the component
repositories; it pins them as submodules at the exact commits produced or used by
the agentic flow.

## Clone

```bash
git clone --recurse-submodules https://github.com/AFOliveira/AGENTIC-DEMO.git
cd AGENTIC-DEMO
```

If submodules are skipped during clone:

```bash
git submodule update --init --recursive
```

## Contents

| Path | Repository | Pinned SHA |
| ---- | ---------- | ---------- |
| `agentic/multiagent` | `AFOliveira/multiagent` | `a21931dfb358b3be373c3be1f84386f27840afbd` |
| `repos/binutils-gdb` | `AFOliveira/binutils-gdb` | `7c075cd6a41256069ce938c2f46a57d1557aa01c` |
| `repos/riscv-unified-db` | `AFOliveira/riscv-unified-db` | `87301e3cf0d5d80991072b23391c9f358353151c` |
| `repos/et-platform` | `AFOliveira/et-platform` | `80bce4f9d91284a470d22163eb14f639a17b594e` |
| `repos/core-et` | `AFOliveira/core-et` | `1a46d2cc9882b8143dba9171b5b65dce5a832aff` |
| `repos/zephyr` | `AFOliveira/zephyr` | `b5813ea96135f7f04f34a485122cd575dc701266` |
| `repos/et-soc1-rtl` | `AFOliveira/et-soc1-rtl` | `529e2b42d95bb3433aa8b8e648f203a1a96de029` |
| `repos/etsoc1-luxonis-experiments` | `AFOliveira/etsoc1-luxonis-experiments` | `91fc1dbcf1cf33ad0347bf4e0347c6ab2d4d218f` |

## Scope

The strict ISA migration branches align UDB, Binutils, `sys_emu`, core-et, and
Zephyr for the demo flow. Current known limitations are recorded in
`MANIFEST.md`; in particular, Zephyr remains compile-only until the patched
Binutils/runtime follow-up is complete.

Note: `et-soc1-rtl` and `etsoc1-luxonis-experiments` are private at the time this
wrapper was created, so anonymous public clones cannot fetch those two
submodules.
