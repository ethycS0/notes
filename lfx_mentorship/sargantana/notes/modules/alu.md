# alu

File: `rtl/datapath/rtl/exe_stage/rtl/alu/alu.sv` (`module alu`, line 21)
Instantiated in [[exe_stage]] `exe_stage.sv:339`.

## Overview

Purely combinational integer ALU for arithmetic, logic, shifts, comparisons,
`SLT`, bit-manipulation and `vsetvl` address formation.

## Ports (alu.sv:24–27)

`instruction_i` (`rr_exe_arith_instr_t`) → `instruction_o`
(`exe_wb_scalar_instr_t`).

## Behaviour

- Sign/zero extension per `instr_type` `:70`–….
- `vsetvl_csr_addr_int` formed from `data_rs2` `:36`.
- Result and metadata assigned to `instruction_o` (valid gated on
  `unit == UNIT_ALU`).

## Related

[[alu]] · [[exe_stage]] · [[drac_pkg]]
