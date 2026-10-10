# rename_table

File: `rtl/datapath/rtl/ir_stage/rtl/rename_table.sv` (`module rename_table`, line 23)
Instantiated in [[datapath]] `datapath.sv:647` (scalar), `:685` (SIMD),
`:727` (FP).

## Overview

Maps architectural register indices to physical register indices. Keeps
`NUM_CHECKPOINTS` (4) speculative copies plus a commit (architectural) copy.

## Ports (rename_table.sv:31–63)

Inputs: `clk_i`, `rstn_i`, `read_src_i[NUM_RENAME_PORTS]`, `old_dst_i`,
`write_dst_i`, `new_dst_i`, `use_rs_i`, `ready_i`, `vaddr_i`, `paddr_i`,
`recover_commit_i`, `commit_old_dst_i`, `commit_write_dst_i`, `commit_new_dst_i`,
`do_checkpoint_i`, `do_recover_i`, `delete_checkpoint_i`, `recover_checkpoint_i`.
Outputs: `src_o`, `rdy_o`, `old_dst_o`, `rdy_old_dst_o`, `checkpoint_o`,
`out_of_checkpoints_o`.

## Behaviour

- Tables: `register_table_*[NUM_ISA_REGISTERS][NUM_CHECKPOINTS]`, `ready_table_*`,
  `commit_table_*` `:124`–`:130`.
- Commit recovery `:139`; checkpoint copy `:169`; write new mapping `:184`.
- Ready propagation across checkpoints `:204`.
- Output registers `:219`; sequential table update `:246`; per-port ready `:271`.
- `out_of_checkpoints_o` `:285`; optional `CHECK_RENAME` `:287`.

## Related

[[rename_table]] · [[ir_stage]] · [[free_list]] · [[regfile]] · [[drac_pkg]]
