# IF Stage (Fetch)

Files:

- `rtl/datapath/rtl/if_stage_1/rtl/if_stage_1.sv`
- `rtl/datapath/rtl/if_stage_2/rtl/if_stage_2.sv`

Wired in [[datapath]] at `datapath.sv:411` (IF1) and `datapath.sv:446` (IF2).

## Purpose

Two-cycle fetch. IF1 maintains the PC and issues the i-cache request; IF2 consumes the i-cache response and produces the 32-bit instruction with its fetch exceptions.

## IF1 (`if_stage_1.sv`)

- `next_pc` selection from `cu_if_t.next_pc` — [[control_unit]] decides between
  keep-PC, predicted/PC+4, jump, debug (`if_stage_1.sv:85`).
- PC register `if_stage_1.sv:123`.
- Exceptions: access fault `:138`, guest page fault `:147`, misaligned `:154`.
- i-cache request `:163`; payload `if_1_if_2_stage_t` `:170`.
- [[branch_predictor]] instance `:177`; prediction packed at `:218`.

## IF2 (`if_stage_2.sv`)

- Exception priority `:74`; instruction select `:105`; valid `:107`; miss stall `:109`.
- Response buffer for stalls `:122`–`:156` (register `resp_icache_cpu_q`).

## Interfaces

- Output: `req_cpu_icache_t` (`req_cpu_icache_o`), input `resp_icache_cpu_t`.
- Stage registers: `reg_if_1_inst` (`datapath.sv:437`), `reg_if_2_inst`
  (`datapath.sv:461`), both flushed by `flush_int.flush_if`.
- Redirect source: `pc_jump_if_int` (`datapath.sv:386`).

## Related

[[if_stage_1]] · [[if_stage_2]] · [[branch_predictor]] · [[bimodal_predictor]] · [[return_address_stack]]
