# ID Stage (Decode)

File: `rtl/datapath/rtl/id_stage/rtl/decoder.sv` (`module decoder`, line 23)
Wired in [[datapath]] at `datapath.sv:476`.

## Purpose

Translate a raw 32-bit instruction into the micro-op struct `instr_entry_t`
(wrapped by `id_ir_stage_t`) that all later stages consume: register addresses,
use flags, immediate, functional unit, control bits and memory metadata.

## Structure

- Ports `decoder.sv:26`–`:54`.
- Combinational default assignment `:103`–…; opcode `case` begins `:195`.
- [[immediate]] instance `:98`.
- [[vset_module]] instance `:3560` (outputs `vl_short_o` `:3588`,
  `prev_vtype_o`, `full_vset_queue_o`).
- JAL/JALR redirect `jal_id_if_o` `:249`; RAS push/pop decisions `:221`–`:247`.
- Illegal / virtual instruction detection spread across the opcode case.

## Outputs

- `decoded_instr` (`id_ir_stage_t`) — see [[drac_pkg]] `instr_entry_t` at
  `drac_pkg.sv:494`.
- `jal_id_if_int` (`jal_id_if_t`) → PC redirect in [[datapath]] `:386`.
- `id_cu_int` assembled in [[datapath]] `datapath.sv:507`–`:520`.

## Stall replay

When ID stalls, the decoded result is kept in `stored_instr_id_q`
(`datapath.sv:544`) and muxed back with `src_select_id_ir_q`
(`datapath.sv:554`, `:562`).

## Related

[[decoder]] · [[immediate]] · [[vset_module]] · [[vset_queue]] · [[drac_pkg]]
