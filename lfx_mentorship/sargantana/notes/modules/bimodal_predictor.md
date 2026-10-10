# bimodal_predictor

File: `rtl/datapath/rtl/if_stage_1/rtl/bimodal_predictor.sv` (`module bimodal_predictor`)
Instantiated in [[branch_predictor]] `branch_predictor.sv:64`.

## Overview

2-bit saturating-counter bimodal predictor with a branch-target buffer.

## Parameters

- `_LENGTH_BIMODAL_INDEX_ = 7` → 128 PHT entries.
- `_BITS_BIMODAL_STATE_MACHINE_ = 2`.

## Behaviour

- Pattern history table read by `pc_fetch_i[8:2]`.
- Branch target buffer read by same index; address sign-extended to `XLEN`.
- State update from `branch_taken_result_exec_i` when `is_branch_EX_i`.

## Related

[[bimodal_predictor]] · [[branch_predictor]] · [[drac_pkg]]
