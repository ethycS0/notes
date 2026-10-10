# mul_unit

File: `rtl/datapath/rtl/exe_stage/rtl/mul_unit.sv` (`module mul_unit`, line 21)
Instantiated in [[exe_stage]] `exe_stage.sv:344`.

## Overview

Pipelined integer multiplier. Latency is tracked by [[score_board_scalar]]
(2 cycles for 32-bit, 3 for 64-bit). Supports all RV64M multiply flavours
including high-half variants.

## Ports (mul_unit.sv:24–30)

`clk_i`, `rstn_i`, `flush_mul_i`, `instruction_i` (`rr_exe_arith_instr_t`);
output `instruction_o` (`exe_wb_scalar_instr_t`).

## Behaviour

- Two pipeline registers (`instruction_0_q`, `instruction_1_q`).
- Operand negation for signed ops `:74`; `same_sign` `:60`.
- 128-bit product and high/low result selection.
- `flush_mul_i` kills in-flight multiply.

## Related

[[mul_unit]] · [[exe_stage]] · [[score_board_scalar]] · [[drac_pkg]]
