# AGENTIC DEMO Manifest

Created: 2026-06-05

## Agentic System

| Component | SHA | Notes |
| --------- | --- | ----- |
| `AFOliveira/multiagent` | `a21931dfb358b3be373c3be1f84386f27840afbd` | Multi-engine worker adapters and resumable sessions. Local tests: `82 passed`. |

## ISA Migration Stack

| Component | SHA | Visibility | Role |
| --------- | --- | ---------- | ---- |
| `AFOliveira/binutils-gdb` | `7c075cd6a41256069ce938c2f46a57d1557aa01c` | Public | Patched XAIFET opcode tables for the strict migration targets. |
| `AFOliveira/riscv-unified-db` | `87301e3cf0d5d80991072b23391c9f358353151c` | Public | UDB encodings for the strict migration targets. |
| `AFOliveira/et-platform` | `80bce4f9d91284a470d22163eb14f639a17b594e` | Public | `sys_emu` strict migration handlers and build aids. |
| `AFOliveira/core-et` | `1a46d2cc9882b8143dba9171b5b65dce5a832aff` | Public | RTL/DV/Verilator gate for `packb`, `bitmixb`, and `aif.europeriscvsummit`. |
| `AFOliveira/zephyr` | `b5813ea96135f7f04f34a485122cd575dc701266` | Public | Strict compile-only Zephyr ISA migration smoke sample and headers. |

## Validation Snapshot

Against the strict branch set, local `aiflow validate --allow-arch-blockers
--allow-pending` completed successfully.

The strict validation report showed:

- 9 proposed entries checked.
- 3 fully matching entries: `packb`, `bitmixb`, `aif.europeriscvsummit`.
- 6 remaining `rtl_pending` entries: `flq2`, `fsq2`, `faddi.pi`, `fandi.pi`,
  `fcmov.ps`, `fcmovm.ps`.
- 2 architecture-decision entries excluded from claims: `fbci.ps`, `fbci.pi`.

## Known Limitations

- Zephyr evidence is compile-only in this pinned snapshot. It does not yet prove
  Zephyr was built through the newly patched Binutils assembler, and it does not
  boot/run the Zephyr ELF on `sys_emu` or full-system Verilator.
- The core-et branch includes focused RTL/DV evidence, but also carries Verilator
  lint suppressions across broader RTL. Treat this as bring-up/demo quality, not
  final clean RTL signoff.
