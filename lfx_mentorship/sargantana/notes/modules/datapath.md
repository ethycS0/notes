# datapath

File: `rtl/datapath/rtl/datapath.sv` (`module datapath`, line 21)

## Overview

Top-level integration of the Sargantana pipeline. It contains **no logic of its
own beyond wiring and small muxes**: it instantiates every stage unit and stage
register and connects them with the struct types from [[drac_pkg]].

## Ports

- Clock/reset, `reset_addr_i`, optional `core_id_i`.
- i-cache: `resp_icache_cpu_i` in, `req_cpu_icache_o` out.
- d-cache: `resp_dcache_cpu_i` in, `req_cpu_dcache_o` out.
- CSR: `resp_csr_cpu_i` in, `req_cpu_csr_o` out.
- Translation/privilege inputs (`en_*_translation_i`, `csr_*_i`).
- Debug ring (`debug_reg_i/o`, `debug_contr_i/o`), VISA (`visa_o`), PMU
  (`pmu_flags_o`), `dtlb_comm_i/o`.

## Internal organisation (block order in file)

1. Signal declarations `:80`–`:334`.
2. [[control_unit]] `:352`.
3. PC jump mux `:386`–`:404`.
4. Fetch: [[if_stage_1]] `:411`, `reg_if_1_inst` `:437`, [[if_stage_2]] `:446`,
   `reg_if_2_inst` `:461`.
5. Decode: [[decoder]] `:476`, `id_cu_int` `:507`.
6. Rename: [[instruction_queue]] `:567`, [[free_list]] ×3 `:579`/`:605`/`:625`,
   [[rename_table]] ×3 `:647`/`:685`/`:727`, IR→RR regs `:786`/`:796`.
7. Read: [[graduation_list]] `:948`, snoop `:982`, [[regfile]] ×3 `:1015`/`:1035`/`:1057`,
   RR→EXE reg `:1128`.
8. Execute: snoop `:1140`, [[exe_stage]] `:1241`, EXE→WB reg `:1338`/`:1347`.
9. Write-back `:1381`/`:1488`.
10. Commit: [[csr_interface]] `:1532`, redirect `:1554`, commit control `:1570`.
11. Sim commit-log / Konata dumps `:1626`/`:1710`; OpenPiton PC FIFO `:1781`.

## Key wires

| Name                                   | Meaning                    |
| -------------------------------------- | -------------------------- |
| `control_int` (`pipeline_ctrl_t`)      | stalls + PC select from CU |
| `flush_int` (`pipeline_flush_t`)       | flush/kill from CU         |
| `stage_if_1_if_2_q`, `stage_if_2_id_q` | fetch registers            |
| `stage_iq_ir_q`, `stage_ir_rr_q`       | rename/read registers      |
| `stage_rr_exe_q`                       | RR→EXE register            |
| `wb_scalar/wb_simd/wb_fp`              | WB payloads                |
| `instruction_to_commit`                | GL commit window           |

## Related

[[datapath]] · [[control_unit]] · [[pipeline_overview]]
