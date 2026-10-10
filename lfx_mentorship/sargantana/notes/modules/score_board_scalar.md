# score_board_scalar

File: `rtl/datapath/rtl/exe_stage/rtl/score_board_scalar.sv` (`module score_board_scalar`, line 21)
Instantiated in [[exe_stage]] `exe_stage.sv:178`.

## Overview

Structural-hazard scoreboard for the scalar pipelines (ALU 1-cycle, MUL 2/3,
DIV 17/32). Produces per-latency `ready` flags and selects one of the two
[[div_unit]] instances.

## Ports (score_board_scalar.sv:24–41)

Inputs: `clk_i`, `rstn_i`, `flush_i`, `set_mul_32_i`, `set_mul_64_i`,
`set_div_32_i`, `set_div_64_i`.
Outputs: `ready_1cycle_o`, `ready_mul_32_o`, `ready_mul_64_o`, `ready_div_32_o`,
`div_unit_sel_o`, `ready_div_unit_o`.

## Behaviour

- Shift register `inst_q[32:0]` marks busy cycles `:52`.
- `set_mul_32` → bit0, `set_mul_64` → bit1, `set_div_32` → bit16,
  `set_div_64` → bit32.
- Two `ocup_div_unit_q` track divider occupancy; `div_unit_sel_o` `:113`.
- `flush_i` clears all occupancy.

## Related

[[score_board_scalar]] · [[exe_stage]] · [[mul_unit]] · [[div_unit]] · [[drac_pkg]]
