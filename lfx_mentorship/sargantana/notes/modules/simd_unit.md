# simd_unit

File: `rtl/datapath/rtl/exe_stage/rtl/simd/simd_unit.sv` (`module simd_unit`, line 21)
Size: ~1972 lines. Instantiated in [[exe_stage]] `exe_stage.sv:371`.

## Overview

Vector/Integer-SIMD execution unit. Splits vector operands into elements, runs a
set of vector functional units, and can also produce FP and scalar results
(vector→scalar reductions, `vmv.x.f`, etc.).

## Ports (simd_unit.sv:24–34)

`clk_i`, `rstn_i`, `flush_i`, `vxrm_i`, `instruction_i`
(`rr_exe_simd_instr_t`); outputs `instruction_scalar_o`
(`exe_wb_scalar_instr_t`), `instruction_simd_o` (`exe_wb_simd_instr_t`),
`stall_prev_o`, `stall_post_o`, `instruction_fp_o` (`exe_wb_fp_instr_t`).

## Behaviour

- Element decomposition into `vs1_elements`/`vs2_elements`, mask handling.
- Contains the vector integer datapath (add/sub, mul, div, shifts, comparisons,
  reductions, iota, …) — see [[simd_subunits]] for the leaf modules.
- `DIV_STAGES = 32`; skip logic for consecutive identical div/rem.
- Interlocks with [[score_board_simd]] via `stall_prev_o`/`stall_post_o` and
  `exe_stages`.

## Related

[[simd_unit]] · [[simd_subunits]] · [[exe_stage]] · [[score_board_simd]] · [[drac_pkg]]
