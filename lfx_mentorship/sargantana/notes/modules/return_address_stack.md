# return_address_stack

File: `rtl/datapath/rtl/if_stage_1/rtl/return_address_stack.sv` (`module return_address_stack`)
Used by [[decoder]] for JAL/JALR RAS push/pop.

## Overview

A 16-entry circular return-address stack (`_LENGTH_RAS_ = 4`) used to predict
function returns.

## Ports

`rstn_i`, `clk_i`, `pc_execution_i`, `push_i`, `pop_i`, `return_address_o`.

## Behaviour

- `output_pointer = head_pointer - 1`.
- Push stores `pc_execution_i` and increments head; pop decrements head;
  simultaneous push+pop overwrites top (`return_address_stack.sv`).

## Related

[[return_address_stack]] · [[branch_predictor]] · [[decoder]]
