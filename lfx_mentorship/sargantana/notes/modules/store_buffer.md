# store_buffer

File: `rtl/datapath/rtl/exe_stage/rtl/store_buffer.sv` (`module store_buffer`)
Used by [[load_store_queue]].

## Overview

Depth-`ST_BUF_NUM_ENTRIES = 8` buffer of committed/pending stores used for
store-to-load forwarding and collision detection.

## Ports (store_buffer.sv:24–42)

Inputs: `clk_i`, `rstn_i`, `write_enable_i`, `instruction_i`, `flush_i`,
`advance_head_i`, `load_addr_i`, `load_size_i`.
Outputs: `finish_instr_o`, `empty_o`, `full_o`, `collision_o`.

## Behaviour

- Circular buffer; `write_enable` requires space.
- `collision_o` compares a new load address/size against buffered stores to force
  a replay (load-after-store hazard).
- `finish_instr_o` returns the next retired store entry.

## Related

[[store_buffer]] · [[load_store_queue]] · [[mem_unit]] · [[drac_pkg]]
