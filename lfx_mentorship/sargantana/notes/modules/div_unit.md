# div_unit

File: `rtl/datapath/rtl/exe_stage/rtl/div_unit.sv` (`module div_unit`, line 21)
Instantiated in [[exe_stage]] `exe_stage.sv:352`.

## Overview

Two interleaved iterative dividers (32-bit: 17 cycles, 64-bit: 32 cycles).
`div_unit_sel_i` picks which physical divider executes the new op; scheduling is
done by [[score_board_scalar]].

## Ports (div_unit.sv:24–31)

`clk_i`, `rstn_i`, `flush_div_i`, `div_unit_sel_i`, `instruction_i`
(`rr_exe_arith_instr_t`); output `instruction_o` (`exe_wb_scalar_instr_t`).

## Behaviour

- Per-divider state arrays `instruction_q[1:0]`, `div_zero_q`, `same_sign_q`,
  `op_32_q`, remainder/quotient/divisor regs.
- Cycle counter per divider `:65`; iterative shift-subtract core.
- Produces quotient and remainder results; `flush_div_i` kills in-flight ops.

## Related

[[div_unit]] · [[exe_stage]] · [[score_board_scalar]] · [[drac_pkg]]
