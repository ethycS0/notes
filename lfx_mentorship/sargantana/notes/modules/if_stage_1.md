# if_stage_1

File: `rtl/datapath/rtl/if_stage_1/rtl/if_stage_1.sv` (`module if_stage_1`, line 22)

## Overview

Fetch stage 1: PC generation and i-cache request. Also hosts the branch predictor.

## Ports (if_stage_1.sv:27–62)

`clk_i`, `rstn_i`, `v_mode_i`, `reset_addr_i`, `stall_i`, `stall_debug_i`,
`cu_if_i` (`cu_if_t`), `invalidate_icache_i`, `invalidate_buffer_i`,
`en_translation_i`, `en_g_translation_i`, `resp_icache_cpu_i`, `pc_jump_i`,
`exe_if_branch_pred_i`, `retry_fetch_i`; outputs `req_cpu_icache_o`,
`fetch_o` (`if_1_if_2_stage_t`), optional `id_o`.

## Behaviour

- `next_pc` mux `:85` driven by `cu_if_i.next_pc` (`next_pc_sel_t`).
- PC register `:123`.
- Fetch exceptions `:138`–`:160`.
- i-cache request `:163`–`:167`; `fetch_o` `:170`.
- [[branch_predictor]] `:177`; prediction packed `:218`.

## Related

[[if_stage_1]] · [[if_stage]] · [[branch_predictor]] · [[datapath]] · [[drac_pkg]]
