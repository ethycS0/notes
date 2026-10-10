# vagu

File: `rtl/datapath/rtl/exe_stage/rtl/vagu.sv` (`module vagu`, line 22)
Instantiated in [[exe_stage]] `exe_stage.sv:458`.

## Overview

Vector Address Generation Unit. Decomposes vector memory operations
(unit-stride, strided, indexed) into element-level addresses, applying the vector
mask and element width, feeding the scalar memory pipeline in [[mem_unit]].

## Parameters

`DCACHE_RESP_DATA_WIDTH` (set to `VLEN` by [[exe_stage]]), `MAX_VELEM`.

## Ports (vagu.sv:29–51)

Inputs: `clk_i`, `rstn_i`, `memp_instr_i` (`rr_exe_mem_instr_t`), `flush_i`,
`stall_i`, `ovi_mask_idx_valid_i`, `ovi_mask_idx_item_i`, `vsew_i`, `mop_i`,
`vl_i`, `stride_i`, `masked_op_i`, `vstore_data_valid_i`, `vstore_data_i`.
Outputs: `stall_o`, `end_o`, `misalign_xcpt_o`, `velem_incr_o`, `velem_id_o`,
`load_mask_o`, `memp_instr_o`.

## Behaviour

- `req_vmem_ops_t` encodes SCALAR / VL_UNIT / VS_UNIT / VL_STRIDED / VS_STRIDED /
  VL_INDEXED / VS_INDEXED.
- Iterates `vl` elements, packing valid elements per cache beat
  (`velem_incr_o`, `load_mask_o`).
- Detects misaligned vector accesses.

## Related

[[vagu]] · [[exe_stage]] · [[mem_unit]] · [[drac_pkg]]
