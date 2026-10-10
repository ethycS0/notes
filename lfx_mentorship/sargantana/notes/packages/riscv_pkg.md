# riscv_pkg

File: `includes/riscv_pkg.sv` (1536 lines).

## Purpose

RISC-V ISA constants and encodings shared by the decoder, immediate generator,
CSR interface and commit.

## Key contents

- `XLEN`, `VLEN`, `INST_SIZE` and related widths.
- Instruction field/opcode enums: `OP_*`, `F3_*`, `F6_*`, `F7_*`.
- `instruction_t` packed union with `common`, `itype`, `stype`, `btype`, `utype`,
  `jtype`, `r4type`, `vtype` views (used by [[immediate]] and [[decoder]]).
- Exception causes (`exception_cause_t`) and CSRs (`SSTATUS_*`, `HSTATUS_*`,
  `MIP_*`).
- FP format/rounding enums `FMT_*`, `op_frm_fp_t`.
- Vector field enums (`F6_VSLL`, `F6_VROR`, `F3_OPIVI`, `F3_OPCFG`, …).

## Related

[[riscv_pkg]] · [[drac_pkg]] · [[def_pkg]] · [[decoder]] · [[immediate]]
