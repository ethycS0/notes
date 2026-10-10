# graduation_list

File: `rtl/datapath/rtl/wb_stage/rtl/graduation_list.sv` (`module graduation_list`, line 21)
Instantiated in [[datapath]] `datapath.sv:948`.

## Overview

In-order reorder buffer of depth `NUM_ENTRIES = 32`. Every renamed instruction
gets a `gl_index` slot; write-back marks the slot valid; commit reads the two
oldest contiguous valid entries. Tracks the oldest exception and CSR address.

## Ports (graduation_list.sv:28–74)

- Instruction in: `instruction_i` (`gl_instruction_t`), `is_csr_i`,
  `is_vector_vl_0_i`, `csr_addr_i`, `ex_i`.
- Read: `read_head_i[1:0]` → `instruction_o[1:0]`, `commit_gl_entry_o`.
- Write-back: scalar/SIMD/FP arrays (`gl_index_t`, enable, `gl_wb_data_t`).
- Exception from EXE: `ex_from_exe_i`, `ex_from_exe_index_i`.
- Flush: `flush_i` + `flush_index_i`, `flush_commit_i`.
- Outputs: `assigned_gl_entry_o`, `full_o`, `empty_o`, `csr_addr_o`,
  `exception_o`, `result_o`, `vsetvl_vtype_o`.

## Behaviour

- `write_enable` `:113`; `read_enable` from commit ports `:117`.
- Entry/valid update `:126`; write-back updates `:137`–`:185`.
- Exception tracking (oldest wins) `:189`–`:213`.
- `result_q`/`vsetvl_vtype_q` capture for CSR/VSETVL `:216`.
- Pointer/`num` update incl. flush rollback `:256`.
- Two-entry commit window `:281`.
- `assigned_gl_entry_o = tail` `:297`; `full_o` at `NUM_ENTRIES-1` `:299`.

## Related

[[graduation_list]] · [[wb_stage]] · [[commit_stage]] · [[drac_pkg]]
