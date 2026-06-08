# Public evidence appendix — clean-release ISA migration

This file summarizes proof boundaries for the pinned component SHAs in this
snapshot. Evidence was produced by a MULTIAGENT clean-release run on a separate
**implementation checkout** (workflow tools, AIFLOW harness, job logs, and
integration bundles). This public demo repository does **not** commit that
harness tree or raw job logs; treat the bullets below as sanitized summaries
with pass tokens a reviewer can match against the pinned forks.

## Source experiment (not fully published here)

| Artifact | SHA / ref | Role |
| -------- | --------- | ---- |
| Implementation experiment root | `97a746dcc8f7f8f8e7f3fb5f2cdb31fa42e98020` on branch `multiagent/clean-release-flow-20260604` | MULTIAGENT task state, integration bundles, harness tooling |
| This public demo snapshot | `004aabbf996219e3366f22f2eee05da332dd6ece` | Agent team + submodule pins only |

Component submodule SHAs in this demo match the experiment root above. Branches
on the public forks are named
`multiagent/isa-migration-clean-release-20260604/isa-migration-release`.

## Targets in scope

Nine **proposed** custom instructions from the clean-release encoding plan.
`fbci.ps` and `fbci.pi` remain **excluded** (`needs_arch_decision`).

## Surface evidence

### UDB and encoding plan

| Check | Pass token / result |
| ----- | ------------------- |
| UDB truth audit | PASS, 9 proposed targets, no blocking gaps |
| Encoding analysis (`aiflow audit-udb`, `report`, `propose`) | PASS, 0 conflicts, 9 remaps confirmed |
| Decodability gate (`check-encodings`) | exit 0, 119 custom instructions, 0 conflicts |
| Manifest validation (`aiflow validate --allow-pending`) | PASS |

### Binutils (patched gas/objdump)

| Check | Pass token / result |
| ----- | ------------------- |
| Round-trip assemble/disassemble | **9/9 PASS** |
| OLD-encoding retirement (migrated targets) | **8/8 PASS** |
| New instruction scaffold | `aif.europeriscvsummit` required manual UDB/Binutils scaffold before `aiflow apply` (generator gap; documented in source experiment) |

Canonical NEW words exercised in toolchain/Zephyr proofs include:
`0x0005500b`, `0x0005502b`, `0x0400802b`, `0x0400902b`, `0x1c20a02b`,
`0x0020802b`, `0x80c5e52b`, `0x80c5f52b`, `0x02a5c52b`.

### sysemu (`sys_emu` strict equivalence)

| Check | Pass token / result |
| ----- | ------------------- |
| Migrated `sys_emu` self-equivalence (Phase 2) | strict equivalence **9/9** |
| Target-to-baseline strict rerun (Phase 3) | **`SYSEMU_STRICT_EQUIV_PASS 9/9`** |
| Direct OLD-word first-illegal negative checks | PASS for migrated targets (e.g. `fsq2` `0x00005027`, `faddi.pi` `0x0400003f`, `packb` `0x8000603b`, summit probe `0x0000402a`) |

**Boundary:** `sys_emu` evidence is instruction dispatch/execute-equivalence on
packaged vectors. It is **not** Zephyr OS boot for `erbium_minion`.

### core-et RTL / Verilator (decoder scope)

Applies to `packb`, `bitmixb`, and `aif.europeriscvsummit` (entries with
`rtl_ctrl`).

| Check | Pass token / result |
| ----- | ------------------- |
| Directed decode DV (`intpipe_decode`) | **TEST PASSED 14/14** (canonical words) |
| Repo lint | **65 passed, 0 failed** |
| Migration rtlcosim vs original decode | **`COSIM PASSED`** (3 comparisons, 0 mismatches; requires `ORIG_ROOT` baseline tree) |

**Boundary:** proves decode-row / control-packet correctness on fixed words, not
full architectural writeback or Zephyr runtime.

### Zephyr (`erbium_minion` / `isa_migration_smoke`)

Pinned fork: `AFOliveira/zephyr` @ `5ccd6f7231e8ef741ecaa803103498b1344f5554`.

| Check | Pass token / result |
| ----- | ------------------- |
| `west build -p always -b erbium_minion samples/aifoundry/isa_migration_smoke` | **PASS** (exit 0) |
| Patched `as-new` / `objdump` mnemonic verification (`CONFIG_AIF_ET_ISA_MIGRATION_PATCHED_ASM=y`) | **9/9 PASS** |
| Non-execution proof | `main()` does not call raw-word or mnemonic emitters; 9/9 NEW words retained in ELF; **0 OLD hits** in executable sections |
| **`erbium_emu` runtime boot** | UART shows Zephyr boot + sample printk (see below) |

**Working runtime command** (C++ Erbium emulator, not Verilator):

```sh
ERBIUM_EMU=repos/et-platform/sw-sysemu/build/erbium_emu
ELF=repos/zephyr/build/zephyr/zephyr.elf
$ERBIUM_EMU -elf_load "$ELF" -reset_pc 0x40000200 \
  -uart_tx_file uart_tx.txt -max_cycles 20000000
```

**Observed UART excerpt** (sanitized):

```text
*** Booting Zephyr OS build 76e134655cec ***
isa-migration-smoke: strict compile-only PASS (no raw insn in main)
  expected packb x10=0x2211 bitmixb x10=0xf895 summit x10=0xaa18
```

Sample README at
`repos/zephyr/samples/aifoundry/isa_migration_smoke/README.md` (in the pinned
Zephyr fork) documents build, runtime, and evidence boundaries.

**Not valid for this board:** booting the same ELF with ET-SoC1 `sys_emu` using
`-shires 0x1` and `-reset_pc 0x40000200` — wrong emulator/memory map for
`erbium_minion`; prior attempt trapped before application output.

**Still blocked:** full-system Verilator load/boot of `zephyr.elf` (no SoC-level
Verilator top + ELF loader + UART capture + Zephyr runner integration in the
public component trees).

## Remaining publication gaps

- This demo snapshot does not commit integration patches, `artifacts/isa-release/`,
  or MULTIAGENT job logs from the source experiment.
- Final **public release integration and push** of all component branches was
  still in progress when this evidence appendix was written; verify fork branch
  heads against the SHAs in `README.md` before treating the demo as a published
  release tag.
- Orchestration prompt fixes (planner-owned dispatch) are tracked separately
  (AQ-P3); this job does not change `.multiagent/**`.

## How to inspect the pinned forks

After `git submodule update --init --recursive`:

```sh
# Example: Zephyr sample docs and runtime notes
sed -n '1,120p' repos/zephyr/samples/aifoundry/isa_migration_smoke/README.md

# Example: et-platform erbium emulator build path (build locally)
ls repos/et-platform/sw-sysemu/build/erbium_emu 2>/dev/null || echo "build erbium_emu in sw-sysemu first"
```

Re-running the full MULTIAGENT pipeline requires the implementation harness
(AIFLOW, workflow scripts, encoding plan, runtime vectors) that is **not** part
of this top-level repository — see `README.md` → **Runnable layout**.
