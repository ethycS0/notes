# pending_mem_req_queue

File: `rtl/datapath/rtl/exe_stage/rtl/pending_mem_req_queue.sv` (`module pending_mem_req_queue`)
Instantiated in [[mem_unit]] `mem_unit.sv:623`.

## Overview

Depth-`PMRQ_NUM_ENTRIES = 16` table of in-flight d-cache requests, used to match
asynchronous cache responses/replays back to their instruction by tag.

## Ports (pending_mem_req_queue.sv:24–43)

Inputs: `clk_i`, `rstn_i`, `instruction_i`, `tag_i`, `replay_valid_i`,
`response_valid_i`, `tag_next_i`, `replay_data_i`, `flush_i`, `advance_head_i`,
`mv_back_tail_i`.
Outputs: `finish_instr_o` (`pmrq_instr_t`), `full_o`.

## Behaviour

- Circular buffer of `pmrq_instr_t`.
- `mv_back_tail_i` supports rollback when a request is killed.
- Response matching on `tag_next_i` produces `finish_instr_o`.

## Related

[[pending_mem_req_queue]] · [[mem_unit]] · [[drac_pkg]]
