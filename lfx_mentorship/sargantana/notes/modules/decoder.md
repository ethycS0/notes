# decoder

File: `rtl/datapath/rtl/id_stage/rtl/decoder.sv` (`module decoder`, line 23)
Size: ~3656 lines. Wired in [[datapath]] `datapath.sv:476`.

## Overview

The instruction decoder is the largest file in the datapath. It is a large
`case` over opcode/funct fields that fills the `instr_entry_t` micro-op.
This note documents its **interface and role**, not its full contents.

## Ports (decoder.sv:26–54)

Inputs: `clk_i`, `rstn_i`, `flush_i`, `stall_i`, `decode_i` (`if_id_stage_t`),
`priv_lvl_i`, `v_mode_i`, `frm_i`, `csr_fs_i`, `csr_vs_i`, `csr_m/s/h_cmo_i`,
`vset_rs1_i`, `vset_rs2_i`, `csr_hu_i`, `write_vset_i`, `commit_vset_i`,
`recover_commit_exception_i`, `recover_last_misspredict_i`,
`vset_index_misspredict_i`, `debug_mode_en_i`.
Outputs: `vl_short_o`, `prev_vtype_o`, `decode_instr_o` (`id_ir_stage_t`),
`jal_id_if_o` (`jal_id_if_t`), `full_vset_queue_o`.

## Implementation reference

- Defaults assigned `:103`–`:192` (all `use_*`, `regfile_we`, `unit = UNIT_ALU`,
  `instr_type = ADD`, `mem_type = NOT_MEM`).
- Main opcode dispatch `case (decode_i.inst.common.opcode)` begins `:195`:
  `OP_LUI`, `OP_AUIPC`, `OP_JAL`, `OP_JALR`, `OP_BRANCH`, `OP_LOAD/STORE`,
  `OP_ALU_I/_W`, `OP_FP*`, `OP_V`, `OP_SYSTEM`, etc.
- [[immediate]] `:98`; [[vset_module]] `:3560`.
- RAS push/pop for JAL/JALR `:221`–`:247`.
- Illegal/`frm`/vector-legal checks inline.

## Role

Produces the fully-annotated instruction consumed by [[ir_stage]]. `jal_id_if_o`
allows the [[control_unit]] to redirect fetch for JAL/JALR that mispredicted.

## Related

[[decoder]] · [[id_stage]] · [[immediate]] · [[vset_module]] · [[drac_pkg]]
