# load_store_queue

File: `rtl/datapath/rtl/exe_stage/rtl/load_store_queue.sv` (`module load_store_queue`)
Instantiated in [[mem_unit]] `mem_unit.sv:297`.

## Overview

Depth-`LSQ_NUM_ENTRIES = 8` queue holding memory instructions until they can be
issued to the d-cache, in order. Integrates the [[store_buffer]] for
store-to-load forwarding and enforces the commit handshake for stores/AMOs.

## Ports (load_store_queue.sv:24–57)

Inputs: `clk_i`, `rstn_i`, `instruction_i`, `en_ld_st_translation_i`,
`en_ld_st_g_translation_i`, `flush_i`, `read_next_i`, `rob_store_ack_i`,
`rob_store_gl_idx_i`, `dtlb_comm_i`, `ld_st_priv_lvl_i`, `ld_st_v_mode_i`.
Outputs: `csr_hs_ld_st_inst_o`, `blocked_store_o`, `next_instr_exe_o`,
`full_o`, `empty_o`, `dtlb_comm_o`, `pmu_load_after_store_o`.

## Behaviour

- Circular buffer of `rr_exe_mem_instr_t`.
- Store-buffer interface (`st_buff_full/empty/collision`) for forwarding and
  load-blocking (`pmu_load_after_store_o`).
- `rob_store_ack_i`/`rob_store_gl_idx_i` gate store issue until commit.

## Related

[[load_store_queue]] · [[mem_unit]] · [[store_buffer]] · [[drac_pkg]]
