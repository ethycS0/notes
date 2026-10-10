# Sargantana Core — Datapath Notes (MOC)

Reference notes for the **Sargantana** in-order, superscalar (2-wide) RISC-V core datapath.
All RTL paths are relative to
`core_tile/rtl/core/sargantana/` unless stated otherwise.

> Source root: `core_tile/rtl/core/sargantana/rtl/datapath/`
> Packages: `core_tile/rtl/core/sargantana/includes/`

## Map of Content

- [[pipeline_overview]] — **start here**. Mermaid block diagram + stage-by-stage breakdown.
- [[datapath]] — top-level `datapath.sv` wiring hub.
- [[control_unit]] — pipeline stalling/flushing/redirect FSM.

### Stages

- [[if_stage]] — Fetch 1 + Fetch 2 (branch prediction, icache handshake).
- [[id_stage]] — Decode (instruction → control, immediate, vset).
- [[ir_stage]] — Instruction queue, free list and register renaming.
- [[rr_stage]] — Physical register read + graduation-list allocation.
- [[exe_stage]] — ALU / MUL / DIV / BRANCH / MEM / SIMD / FPU execution.
- [[wb_stage]] — Write-back muxing + graduation list.
- [[commit_stage]] — In-order retire, CSR, exceptions, PC redirect.

### Modules

- [[datapath]], [[control_unit]]
- Fetch: [[if_stage_1]], [[if_stage_2]], [[branch_predictor]], [[bimodal_predictor]], [[return_address_stack]]
- Decode: [[decoder]], [[immediate]], [[vset_module]], [[vset_queue]]
- Rename: [[instruction_queue]], [[free_list]], [[rename_table]]
- Register read: [[regfile]]
- Execute: [[exe_stage]], [[alu]], [[mul_unit]], [[div_unit]], [[branch_unit]], [[mem_unit]], [[load_store_queue]], [[store_buffer]], [[pending_mem_req_queue]], [[vagu]], [[simd_unit]], [[fpu_drac_wrapper]], [[pending_fp_ops_queue]], [[score_board_scalar]], [[score_board_simd]]
- Commit: [[graduation_list]], [[csr_interface]]

### Packages

- [[drac_pkg]] — configuration, structs and enums for every stage boundary.
- [[def_pkg]] — ISA/extension configuration and exception type.
- [[riscv_pkg]] — instruction encodings, CSRs, causes.

## Quick pipeline summary

| #   | Stage  | Cycle      | Main module(s)                                   | Key file                                              |
| --- | ------ | ---------- | ------------------------------------------------ | ----------------------------------------------------- |
| 1   | IF1    | fetch      | `if_stage_1` + `branch_predictor`                | `rtl/datapath/rtl/if_stage_1/rtl/if_stage_1.sv`       |
| 2   | IF2    | fetch      | `if_stage_2`                                     | `rtl/datapath/rtl/if_stage_2/rtl/if_stage_2.sv`       |
| 3   | ID     | decode     | `decoder`, `immediate`, `vset_module`            | `rtl/datapath/rtl/id_stage/rtl/decoder.sv`            |
| 4   | IQ/IR  | rename     | `instruction_queue`, `free_list`, `rename_table` | `rtl/datapath/rtl/ir_stage/rtl/rename_table.sv`       |
| 5   | RR     | read       | `regfile` + `graduation_list` alloc              | `rtl/datapath/rtl/rr_stage/rtl/regfile.sv`            |
| 6   | EXE    | execute    | `exe_stage` + functional units                   | `rtl/datapath/rtl/exe_stage/rtl/exe_stage.sv`         |
| 7   | WB     | write-back | `graduation_list`                                | `rtl/datapath/rtl/wb_stage/rtl/graduation_list.sv`    |
| 8   | Commit | retire     | `csr_interface`                                  | `rtl/datapath/rtl/interface_csr/rtl/csr_interface.sv` |

> Architecture facts: 2 instructions/cycle fetch & commit, 64 physical registers per class
> (scalar/vector/FP), 4 checkpoints for speculative rename, 32-entry graduation list,
> 16-entry instruction queue. See [[drac_pkg]].
