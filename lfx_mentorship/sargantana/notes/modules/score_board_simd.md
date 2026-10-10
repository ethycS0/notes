# score_board_simd

File: `rtl/datapath/rtl/exe_stage/rtl/score_board_simd.sv` (`module score_board_simd`, line 21)
Instantiated in [[exe_stage]] `exe_stage.sv:195`.

## Overview

Structural-hazard scoreboard for the vector/SIMD pipeline. Tracks multi-cycle
vector operations and reports the number of execution stages to
[[simd_unit]].

## Ports (score_board_simd.sv:21–39)

`clk_i`, `rstn_i`, `flush_i`, `ready_i`, `instr_entry_i` (`rr_exe_simd_instr_t`),
`sew_i`, `vl_i`, `stall_from_vfp`; outputs `simd_exe_stages_o[5:0]`,
`stall_simd_o`.

## Behaviour

- Computes busy window from SEW/VL and instruction type.
- `simd_exe_stages_o` feeds `simd_instr.exe_stages` in [[exe_stage]]
  (`exe_stage.sv:321`).
- `stall_simd_o` blocks new vector issue while busy.

## Related

[[score_board_simd]] · [[exe_stage]] · [[simd_unit]] · [[drac_pkg]]
