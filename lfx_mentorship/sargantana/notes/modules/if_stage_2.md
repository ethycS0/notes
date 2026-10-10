# if_stage_2

File: `rtl/datapath/rtl/if_stage_2/rtl/if_stage_2.sv` (`module if_stage_2`, line 22)

## Overview

Fetch stage 2: aligns the i-cache response with the PC from IF1, resolves fetch
exceptions and emits the instruction word to decode.

## Ports (if_stage_2.sv:25–42)

`clk_i`, `rstn_i`, `v_mode_i`, `en_translation_i`, `en_g_translation_i`,
`fetch_i` (`if_1_if_2_stage_t`), `resp_icache_cpu_i`, `stall_i`, `flush_i`;
outputs `fetch_o` (`if_id_stage_t`), `stall_o`.

## Behaviour

- Exception priority: propagated IF1 exception → instr page fault → guest page
  fault (`:74`–`:102`).
- Instruction select from current/buffered response `:105`.
- `fetch_o.valid` `:107`; icache-miss `stall_o` `:109`.
- Response buffer `resp_icache_cpu_q` for stalls `:122`–`:156`.

## Related

[[if_stage_2]] · [[if_stage]] · [[datapath]] · [[drac_pkg]]
