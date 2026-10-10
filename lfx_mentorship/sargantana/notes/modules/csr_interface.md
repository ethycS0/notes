# csr_interface

File: `rtl/datapath/rtl/interface_csr/rtl/csr_interface.sv` (`module csr_interface`, line 23)
Instantiated in [[datapath]] `datapath.sv:1532`.

## Overview

Bridges commit to the CSR block (`csr_bsc`). Builds `req_cpu_csr_t`, decides
retirement, forwards exceptions/interrupts and FP/vector modification flags.

## Ports (csr_interface.sv:25–47)

Inputs: `commit_xcpt_i`, `result_gl_i`, `csr_addr_gl_i`, `vsetvl_vtype_i`,
`vleff_vl_i`, `instruction_to_commit_i[1:0]`, `stall_exe_i`,
`commit_store_or_amo_i`, `mem_commit_stall_i`, `exception_mem_commit_i`,
`exception_gl_i`, `debug_pc_valid_i`, `debug_pc_i`, `debug_mode_en_i`.
Outputs: `csr_ena_int_o`, `req_cpu_csr_o` (`req_cpu_csr_t`), `retire_inst_o[1:0]`.

## Behaviour

- CSR command decode `:58`–`:140` (CSRRx/I, system, VSETVL\*, VLEFF).
- `commit_2_blocked` serialization check `:159`.
- Request assembly `:190`–`:225`.
- Retirement decision `:211` (1 or 2 instructions; exceptions retire alone).
- Exception cause/origin forwarding `:227`.
- `freg_modified` / `vreg_modified` `:236`–`:238`.

## Related

[[csr_interface]] · [[commit_stage]] · [[graduation_list]] · [[drac_pkg]]
