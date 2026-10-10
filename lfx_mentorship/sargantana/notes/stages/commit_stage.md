# Commit Stage (Retire)

File: `rtl/datapath/rtl/interface_csr/rtl/csr_interface.sv`
Wired in [[datapath]] at `datapath.sv:1532`.

## Purpose

In-order retirement of instructions from the [[graduation_list]]: issue CSR
commands, take exceptions/interrupts, and free old physical registers / update
the commit rename table.

## Data

- `instruction_to_commit = instruction_gl_commit` (`datapath.sv:1526`) — the two
  oldest entries of [[graduation_list]].
- `commit_cu_int` assembled `datapath.sv:1570`–`:1624`: validity, `regfile_we`
  arrays, `csr_enable`, `xcpt`, `ecall_taken`, `fence`, `write_enable`,
  `stall_commit`, `retire`.
- `commit_store_or_amo_int` gates stores/AMOs to commit before issue
  (`datapath.sv:1610`).

## CSR interface

[[csr_interface]] builds `req_cpu_csr_t` for the CSR block (`csr_interface.sv:58`),
decides retirement (`:211`), and drives `retire_inst_o`.

## Exceptions & redirect

- `commit_xcpt` mux between GL exception and MEM-commit exception
  (`datapath.sv:1566`).
- Redirect PC registers `pc_evec_q` / `pc_next_csr_q` (`datapath.sv:1554`) feed
  IF1 through `pc_jump_if_int` (`datapath.sv:386`).

## Commit updates

- Free list: `instruction_to_commit[*].old_prd/old_pvd/old_fprd`
  (`datapath.sv:588`, `:610`, `:634`).
- Rename commit table: `commit_old_dst_i` / `commit_new_dst_i`
  (`datapath.sv:667`, `:701`, `:747`).

## Related

[[commit_stage]] · [[csr_interface]] · [[graduation_list]] · [[free_list]] · [[rename_table]]
