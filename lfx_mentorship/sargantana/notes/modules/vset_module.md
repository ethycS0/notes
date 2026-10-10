# vset_module

File: `rtl/datapath/rtl/id_stage/rtl/vset_module.sv` (`module vset_module`, line 21)
Instantiated in [[decoder]] `decoder.sv:3560`.

## Overview

Computes the vector `vl`/`vtype` for `vsetvl*` instructions and tracks the
speculative vector-configuration state via [[vset_queue]].

## Ports (vset_module.sv:24–51)

Inputs: `clk_i`, `rstn_i`, `vtype_i`, `avl_value_i`, `is_vset_i`, `rw_cmd_i`,
`write_vset_i`, `vset_commited_i`, `recover_commit_exception_i`,
`recover_last_misspredict_i`, `vset_index_misspredict_i`.
Outputs: `vset_index_o`, `vnarrow_wide_o`, `vl_o`, `vl_short_o`, `sew_o`,
`vill_o`, `vma_o`, `vta_o`, `vlmul_o`, `vlmax_o`, `prev_vtype_o`,
`full_vset_queue_o`.

## Behaviour

- `vlmax` per LMUL `:65`; `vl` clamping to AVL vs VLMAX `:72`–`:86`.
- Illegal vtype → `vill` and `vl = 0` `:89`.
- Narrowing/wide detection `:98`.
- [[vset_queue]] instance `:110`; final `vl_o`/`vl_short_o` `:140`–`:141`.

## Related

[[vset_module]] · [[vset_queue]] · [[decoder]] · [[drac_pkg]]
