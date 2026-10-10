# EXE Stage (Execute)

File: `rtl/datapath/rtl/exe_stage/rtl/exe_stage.sv` (`module exe_stage`, line 21)
Wired in [[datapath]] at `datapath.sv:1241`.

## Purpose

Single issue point that dispatches to a functional unit based on
`instr.unit` (`functional_unit_t`, [[drac_pkg]] `drac_pkg.sv:317`), resolves
branches, produces memory requests and returns results for write-back.

## Dispatch / readiness

- Operand source select for `use_pc` / `use_imm` (`exe_stage.sv:175`–`:176`).
- [[score_board_scalar]] (`:178`) and [[score_board_simd]] (`:195`).
- Global `ready` (`:209`) combines all `rdy*` bits.
- Instruction-valid gating per unit (`:323`–`:337`).

## Functional units

| Unit       | Instance | File                 |
| ---------- | -------- | -------------------- |
| ALU        | `:339`   | [[alu]]              |
| Mul        | `:344`   | [[mul_unit]]         |
| Div        | `:352`   | [[div_unit]]         |
| Branch     | `:361`   | [[branch_unit]]      |
| SIMD       | `:371`   | [[simd_unit]]        |
| Vector AGU | `:458`   | [[vagu]]             |
| Memory     | `:497`   | [[mem_unit]]         |
| FPU        | `:544`   | [[fpu_drac_wrapper]] |

## Stall / exception / branch

- Per-unit stall aggregation `:597`–`:639`; `exe_cu_o.stall` `:731`.
- Exception prioritisation (MEM older than branch) `:642`.
- Branch-prediction correctness `:674`; forwarding to [[branch_predictor]] `:701`.
- `exe_cu_o.clear_vset_fence` `:732`.

## WB result muxing

`exe_stage.sv:555`–`:595` selects scalar/SIMD/FP write-back payloads
(`exe_wb_*_instr_t`). FP wins over SIMD on the shared scalar port (`:586`).

## Registers

- EXE→WB `reg_exe_inst` (`datapath.sv:1338`/`:1347`). With
  `REGISTER_HPDC_OUTPUT`, MEM scalar/FP results bypass this register
  (`datapath.sv:1287`–`:1295`).
- Branch redirect latched in `branch_addr_result_wb` (`datapath.sv:1357`).

## Related

[[exe_stage]] · [[alu]] · [[mul_unit]] · [[div_unit]] · [[branch_unit]] · [[mem_unit]] · [[simd_unit]] · [[fpu_drac_wrapper]] · [[score_board_scalar]] · [[score_board_simd]]
