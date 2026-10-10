# instruction_queue

File: `rtl/datapath/rtl/ir_stage/rtl/instruction_queue.sv` (`module instruction_queue`, line 21)
Instantiated in [[datapath]] `datapath.sv:567`.

## Overview

Decoupling FIFO between [[decoder]] and rename. Circular buffer of
`id_ir_stage_t`, depth `INSTRUCTION_QUEUE_NUM_ENTRIES = 16` ([[drac_pkg]]
`drac_pkg.sv:104`).

## Ports (instruction_queue.sv:23–34)

`clk_i`, `rstn_i`, `instruction_i`, `flush_i`, `read_head_i`; outputs
`instruction_o`, `full_o`, `empty_o`.

## Behaviour

- `write_enable = valid & num < DEPTH` `:59`.
- `read_enable = read_head_i & num > 0` `:63`.
- Storage array `:66`; pointer/num update `:81`; `flush_i` clears.
- `instruction_o = empty ? '0 : buffer[head]` `:102`.

## Related

[[instruction_queue]] · [[ir_stage]] · [[decoder]] · [[rename_table]] · [[drac_pkg]]
