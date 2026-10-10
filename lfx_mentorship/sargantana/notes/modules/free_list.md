# free_list

File: `rtl/datapath/rtl/ir_stage/rtl/free_list.sv` (`module free_list`, line 21)
Instantiated in [[datapath]] `datapath.sv:579` (scalar), `:605` (SIMD),
`:625` (FP).

## Overview

Checkpointed circular buffer that hands out free physical registers and reclaims
them at commit. Parameterised by `NUM_ENTRIES`, `ZERO_IS_FREEABLE`, `reg_type`.

## Ports (free_list.sv:29–46)

Inputs: `clk_i`, `rstn_i`, `read_head_i`, `add_free_register_i[1:0]`,
`free_register_i[1:0]`, `do_checkpoint_i`, `do_recover_i`,
`delete_checkpoint_i`, `recover_checkpoint_i`, `commit_roll_back_i`.
Outputs: `new_register_o`, `checkpoint_o`, `out_of_checkpoints_o`, `empty_o`.

## Behaviour

- `NUM_CHECKPOINTS = 4` independent head copies `:69`.
- `checkpoint_enable` `:94`; `read_enable` `:112`.
- Write freed regs to `register_table` `:155`; update `tail`/counts `:171`.
- Recover checkpoint `:183`; copy-on-checkpoint `:215`.
- `new_register_o` mux `:227`; `empty_o` `:228`; `out_of_checkpoints_o` `:229`.
- Optional `CHECK_RENAME` self-check `:231`.

## Related

[[free_list]] · [[ir_stage]] · [[rename_table]] · [[graduation_list]] · [[drac_pkg]]
