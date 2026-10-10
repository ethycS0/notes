# fpu_drac_wrapper

File: `rtl/datapath/rtl/exe_stage/rtl/old_fpu/fpu_drac_wrapper.sv`
Instantiated in [[exe_stage]] `exe_stage.sv:544`.

## Overview

Wrapper around the FPU core (FPnew-based). Decodes the FP operation, formats
operands, queues multi-cycle FP ops in [[pending_fp_ops_queue]], and produces FP
and scalar (FP→int) write-back payloads.

## Ports

`rstn_i`, `flush_i`, `stall_wb_i`, `instruction_i` (`rr_exe_fpu_instr_t`);
outputs `instruction_o` (`exe_wb_fp_instr_t`), `instruction_scalar_o`
(`exe_wb_scalar_instr_t`), `stall_o`.

## Behaviour

- Format selection (`FMT_S/D/H` → `FP32/64/16`) and int format.
- Operation decode from `instr_type` (add/mul/fma/div/sqrt/cmp/conv/…).
- Uses `fpnew_pkg::status_t` for flags; tag-based completion via
  [[pending_fp_ops_queue]].
- `stall_wb_i` resolves the shared scalar/FP write-back port conflict
  (see [[exe_stage]] `stall_fpu_wb`).

## Related

[[fpu_drac_wrapper]] · [[exe_stage]] · [[pending_fp_ops_queue]] · [[drac_pkg]]
