# branch_unit

File: `rtl/datapath/rtl/exe_stage/rtl/branch_unit.sv` (`module branch_unit`, line 21)
Instantiated in [[exe_stage]] `exe_stage.sv:361`.

## Overview

Resolves conditional branches and JAL/JALR: computes target, taken decision and
misaligned-target exceptions. Its result trains [[branch_predictor]] and drives
the redirect in [[control_unit]].

## Ports (branch_unit.sv:24–31)

`en_translation_i`, `en_g_translation_i`, `priv_lvl_i`, `v_mode_i`,
`instruction_i` (`rr_exe_arith_instr_t`) → `instruction_o`
(`exe_wb_scalar_instr_t`).

## Behaviour

- Comparison primitives `equal/less/less_u` `:48`–`:50`.
- Target computation `:58` (JAL/JALR mask bit 0; branches `pc+imm`).
- Taken decision per `instr_type` `:78`.
- Metadata + result `pc+4` / `result_pc = target` `:139`–`:141`.
- Misaligned-target exception `:145`–`:169`.

## Related

[[branch_unit]] · [[exe_stage]] · [[branch_predictor]] · [[control_unit]] · [[drac_pkg]]
