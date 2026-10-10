# pending_fp_ops_queue

File: `rtl/datapath/rtl/exe_stage/rtl/pending_fp_ops_queue.sv` (`module pending_fp_ops_queue`)
Used by [[fpu_drac_wrapper]].

## Overview

Depth-`PFPQ_NUM_ENTRIES = 8` queue of in-flight FP operations. Assigns a tag to
each op and matches the asynchronous FP result back to the originating
instruction.

## Ports (pending_fp_ops_queue.sv:24–43)

Inputs: `clk_i`, `rstn_i`, `flush_i`, `valid_i`, `instruction_i`
(`rr_exe_fpu_instr_t`), `result_valid_i`, `result_tag_i`, `result_data_i`,
`result_fp_status_i`, `advance_head_i`.
Outputs: `finish_instr_fp_o` (`rr_exe_fpu_instr_t`), `finish_fp_status_o`,
`tag_o`, `full_o`.

## Behaviour

- `tag_int` counter generates `tag_o` `:52`.
- Circular buffer; `write_enable = valid_i & valid & num < DEPTH`.
- Result matching by `result_tag_i`; `finish_instr_fp_o` returns the completed
  op with its `fp_status`.

## Related

[[pending_fp_ops_queue]] · [[fpu_drac_wrapper]] · [[exe_stage]] · [[drac_pkg]]
