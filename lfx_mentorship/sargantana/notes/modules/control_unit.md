# control_unit

File: `rtl/control_unit/rtl/control_unit.sv` (`module control_unit`, line 21)
Instantiated in [[datapath]] `datapath.sv:352`.

## Overview

Central pipeline controller. Consumes per-stage status structs and produces
stall, flush and redirect control. Despite living outside the `datapath/`
folder it is part of the datapath control loop.

## Ports (control_unit.sv:21–60)

Inputs:

- `miss_icache_i`, `ready_icache_i`, `if2_cu_valid_i`.
- `id_cu_i`, `ir_cu_i`, `rr_cu_i`, `exe_cu_i`, `wb_cu_i`, `commit_cu_i`.
- `csr_cu_i` (`resp_csr_cpu_t`), `correct_branch_pred_exe_i`,
  `correct_branch_pred_wb_i`, `debug_wr_valid_i`, `gl_empty_i`,
  `debug_contr_i`.

Outputs:

- `pipeline_ctrl_o` (`pipeline_ctrl_t`) — per-stage stalls + `sel_addr_if`.
- `pipeline_flush_o` (`pipeline_flush_t`) — per-stage flush + `kill_exe`.
- `cu_if_o`, `cu_ir_o`, `cu_rr_o`, `cu_wb_o`, `cu_commit_o`.
- `invalidate_icache_o`, `invalidate_buffer_o`.
- `debug_contr_o`, `debug_csr_halt_ack_o`, `pmu_jump_misspred_o`.

## Role in the pipeline

- Stalls: `stall_if_1/2`, `stall_id`, `stall_iq`, `stall_ir`, `stall_rr`,
  `stall_exe`, `stall_commit` (see [[drac_pkg]] `pipeline_ctrl_t` `drac_pkg.sv:1018`).
- Flushes: `flush_if/id/ir/rr/exe`, `kill_exe` (`drac_pkg.sv:1033`).
- Redirect: `sel_addr_if` selects among `SEL_JUMP_EXECUTION`, `SEL_JUMP_CSR`,
  `SEL_JUMP_CSR_RW`, `SEL_JUMP_DECODE` (`jump_addr_fetch_t`, `drac_pkg.sv:232`).
- Checkpoint management is forwarded to rename via `cu_ir_int` (`cu_ir_t`).

## Related

[[control_unit]] · [[datapath]] · [[drac_pkg]] · [[pipeline_overview]]
