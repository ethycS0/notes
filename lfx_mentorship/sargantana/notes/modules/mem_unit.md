# mem_unit

File: `rtl/datapath/rtl/exe_stage/rtl/mem_unit.sv` (`module mem_unit`, line 21)
Instantiated in [[exe_stage]] `exe_stage.sv:497`.

## Overview

Memory execution unit: load/store/AMO/CMO pipeline plus vector memory handling.
Owns the [[load_store_queue]] and the [[pending_mem_req_queue]], and drives the
d-cache request port. Split into a 0/1/2-stage state machine (with
`REGISTER_HPDC_OUTPUT` variants).

## Ports (mem_unit.sv:29–70)

Inputs: `clk_i`, `rstn_i`, `kill_i`, `flush_i`, `en_ld_st_translation_i`,
`en_ld_st_g_translation_i`, `instruction_i` (`rr_exe_mem_instr_t`),
`resp_dcache_cpu_i`, `commit_store_or_amo_i`, `commit_store_or_amo_gl_idx_i`,
`dtlb_comm_i`, `ld_st_priv_lvl_i`, `ld_st_v_mode_i`.
Outputs: `csr_hs_ld_st_inst_o`, `req_cpu_dcache_o`, `instruction_scalar_o`,
`instruction_simd_o`, `instruction_fp_o`, `exception_mem_commit_o`,
`mem_commit_stall_o`, `mem_store_or_amo_o`, `mem_gl_index_o`, `lock_o`,
`empty_o`, `dtlb_comm_o`, `vleff_vl_o`, PMU flags.

## Structure

- [[load_store_queue]] instance `:297`.
- State machine `:321`–`:410` (`ResetState`, `ReadHead`, …); request generation
  and commit-store handshake `:372`–`:409`.
- [[pending_mem_req_queue]] instance `:623`; response/tag matching and replay
  `:617`–`:644`, write-back decision `:657`.
- Data extraction per access size `:646`–`:653`.

## Related

[[mem_unit]] · [[exe_stage]] · [[load_store_queue]] · [[store_buffer]] · [[pending_mem_req_queue]] · [[vagu]]
