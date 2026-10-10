# vset_queue

File: `rtl/datapath/rtl/id_stage/rtl/vset_queue.sv` (`module vset_queue`)
Instantiated in [[vset_module]] `vset_module.sv:110`.

## Overview

Circular buffer (`VSET_QUEUE_NUM_ENTRIES = 8`) holding uncommitted vector
configurations (`{vl, vtype, vnarrow_wide_en, vlmax}`). Supports commit,
mispredict recovery and exception recovery so vector state stays precise.

## Ports (vset_queue.sv:24–53)

Inputs: `clk_i`, `rstn_i`, `commited_vset_i`, `new_vset_i`,
`recover_last_committed_i`, `recover_last_misspredict_i`,
`vset_index_misspredict_i`, `vl_i`, `vtype_i`, `vnarrow_wide_en_i`, `vlmax_i`.
Outputs: `vset_index_o`, `sew_o`, `vl_o`, `vnarrow_wide_en_o`, `vill_o`,
`vta_o`, `vma_o`, `vlmul_o`, `vlmax_o`, `prev_vtype_o`, `full_o`.

## Behaviour

Maintains `last_committed_vset_index`, `first_not_committed_index`, `tail`.
`prev_vtype_o` is the first uncommitted vtype, used by [[csr_interface]] for
VSETVL commit.

## Related

[[vset_queue]] · [[vset_module]] · [[decoder]] · [[drac_pkg]]
