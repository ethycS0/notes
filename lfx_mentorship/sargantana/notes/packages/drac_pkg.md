# drac_pkg

File: `includes/drac_pkg.sv` (1800 lines). Package imported by every datapath
module.

## Purpose

Defines the global configuration record, all stage-boundary payload structs,
control structs, enums and derived constants.

## Key configuration

- `drac_cfg_t` `:156` — memory-map and cache configuration; default
  `DracDefaultConfig` `:1292`.
- Address/size constants `:32`–`:149`.
- Register counts: `NUM_PHISICAL_REGISTERS = 64` `:107`,
  `NUM_PHISICAL_VREGISTERS = 64` `:112`,
  `NUM_PHYSICAL_FREGISTERS = 64` `:117`.
- Write-back widths `:63`: `NUM_SCALAR_WB = 4`, `NUM_FP_WB = 2`, `NUM_SIMD_WB = 2`.
- Queue depths: `INSTRUCTION_QUEUE_NUM_ENTRIES = 16` `:104`,
  `VSET_QUEUE_NUM_ENTRIES = 8` `:69`, `LSQ_NUM_ENTRIES = 8` `:136`,
  `PMRQ_NUM_ENTRIES = 16` `:139`, `ST_BUF_NUM_ENTRIES = 8` `:143`,
  `NUM_CHECKPOINTS = 4` `:122`.
- `VLEN`/`VMAXELEM` vector sizing `:44`–`:46`.

## Important structs

| Struct                                   | Line          | Boundary       |
| ---------------------------------------- | ------------- | -------------- |
| `exception_t`                            | `:272`        | anywhere       |
| `branch_pred_t`                          | `:266`        | fetch          |
| `exe_if_branch_pred_t`                   | `:281`        | EXE → fetch    |
| `resp_icache_cpu_t` / `req_cpu_icache_t` | `:291`/`:300` | icache         |
| `resp_dcache_cpu_t` / `req_cpu_dcache_t` | `:446`/`:456` | dcache         |
| `if_1_if_2_stage_t`                      | `:470`        | IF1→IF2        |
| `if_id_stage_t`                          | `:482`        | IF2→ID         |
| `instr_entry_t`                          | `:494`        | micro-op       |
| `id_ir_stage_t`                          | `:557`        | ID→IR          |
| `ir_rr_stage_t`                          | `:562`        | IR→RR          |
| `rr_exe_instr_t`                         | `:594`        | RR→EXE         |
| `rr_exe_arith_instr_t`                   | `:635`        | ALU/MUL/DIV/BR |
| `rr_exe_mem_instr_t`                     | `:654`        | MEM            |
| `rr_exe_simd_instr_t`                    | `:747`        | SIMD           |
| `rr_exe_fpu_instr_t`                     | `:775`        | FPU            |
| `exe_wb_scalar_instr_t`                  | `:796`        | EXE→WB scalar  |
| `exe_wb_simd_instr_t`                    | `:831`        | EXE→WB SIMD    |
| `exe_wb_fp_instr_t`                      | `:866`        | EXE→WB FP      |
| `gl_instruction_t`                       | `:1238`       | GL entry       |
| `gl_wb_data_t`                           | `:1278`       | GL write-back  |
| `commit_data_t`                          | `:1797`       | commit log     |

## Control structs

- `id_cu_t` `:909`, `ir_cu_t` `:919`, `rr_cu_t` `:929`.
- `cu_if_t` `:933`, `cu_ir_t` `:937`, `cu_rr_t` `:948`, `cu_wb_t` `:1012`,
  `cu_commit_t` `:1006`.
- `exe_cu_t` `:959`, `wb_cu_t` `:972`, `commit_cu_t` `:988`.
- `pipeline_ctrl_t` `:1018`, `pipeline_flush_t` `:1033`.
- `to_PMU_t` `:1050`, `req_cpu_csr_t` `:1070`, `resp_csr_cpu_t` `:1098`.
- Debug structs `:1138`–`:1225`.

## Enums

`next_pc_sel_t` `:215`, `mem_type_t` `:223`, `jump_addr_fetch_t` `:232`,
`branch_pred_decision_t` `:247`, `sew_t` `:252`, `alu_sel_t` `:308`,
`functional_unit_t` `:317`, `reg_sel_t` `:329`, `regfile_sel_t` `:336`,
`instr_type_t` `:342`, `csr_cmd_t` `:431`.

## Related

[[drac_pkg]] · [[def_pkg]] · [[riscv_pkg]] · [[pipeline_overview]]
