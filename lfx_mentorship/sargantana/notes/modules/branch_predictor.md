# branch_predictor

File: `rtl/datapath/rtl/if_stage_1/rtl/branch_predictor.sv` (`module branch_predictor`, line 29)
Instantiated in [[if_stage_1]] `if_stage_1.sv:177`.

## Overview

Top of the prediction unit: combines a [[bimodal_predictor]] (direction + target)
with an "is-branch" tag table, and is trained from EXE results.

## Ports (branch_predictor.sv:32–44)

Inputs: `clk_i`, `rstn_i`, `pc_fetch_i`, `pc_execution_i`,
`branch_addr_result_exec_i`, `branch_taken_result_exec_i`, `is_branch_EX_i`.
Outputs: `branch_predict_is_branch_o`, `branch_predict_taken_o`,
`branch_predict_addr_o`.

## Behaviour

- [[bimodal_predictor]] `:64` supplies taken/target.
- 128-entry `is_branch_table` keyed by PC bits `[8:2]` (`:80`, `:103`), trained on
  `is_branch_EX_i` (`:107`).
- Outputs: `branch_predict_addr_o = bimodal_predict_addr` `:126`,
  `branch_predict_taken_o = bimodal_predict_taken` `:129`,
  `branch_predict_is_branch_o = is_branch_prediction` `:132`.

## Related

[[branch_predictor]] · [[bimodal_predictor]] · [[return_address_stack]] · [[if_stage_1]]
