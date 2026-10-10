# IR Stage (Instruction Queue + Rename)

Files:

- `rtl/datapath/rtl/ir_stage/rtl/instruction_queue.sv`
- `rtl/datapath/rtl/ir_stage/rtl/free_list.sv`
- `rtl/datapath/rtl/ir_stage/rtl/rename_table.sv`

Wired in [[datapath]]: IQ `datapath.sv:567`, free lists `:579`/`:605`/`:625`,
rename tables `:647`/`:685`/`:727`.

## Purpose

Buffer decoded instructions, allocate a fresh physical destination register for
each writer, and remap architectural source registers to their physical names.
Renaming is speculative with up to `NUM_CHECKPOINTS = 4` checkpoints
(see [[drac_pkg]]).

## Instruction queue

[[instruction_queue]] is a 16-entry circular buffer of `id_ir_stage_t`. See
`instruction_queue.sv:59` (write), `:63` (read), `:102` (output).

## Free list

[[free_list]] is a checkpointed circular buffer of free physical registers.
One instance per register class:

- scalar: `NUM_ENTRIES = NUM_PHISICAL_REGISTERS - NUM_ISA_REGISTERS`,
  `ZERO_IS_FREEABLE = 0` (`datapath.sv:579`).
- SIMD: `ZERO_IS_FREEABLE = 1` (`datapath.sv:605`).
- FP: `ZERO_IS_FREEABLE = 1` (`datapath.sv:625`).

Checkpoint/rollback inputs come from `cu_ir_int` ([[drac_pkg]] `cu_ir_t`).

## Rename table

[[rename_table]] keeps per-checkpoint copies of the arch→phys map. Scalar has
`NUM_RENAME_PORTS = 2`, SIMD/FP have 3. Sources are read from `stage_iq_ir_q`
(`datapath.sv:654`, `:688`, `:734`).

## IR→RR register

`reg_ir_inst` (`datapath.sv:786`) packs
`{id_ir_stage_t, prd, fprd, pvd, checkpoint_done}` into `ir_rr_stage_t`.
A shadow register `reg_rename_inst` (`datapath.sv:796`) plus mux
(`datapath.sv:806`–`:872`) replays the rename one cycle when stalled, and merges
WB snoops into the ready bits.

## Related

[[ir_stage]] · [[instruction_queue]] · [[free_list]] · [[rename_table]] · [[drac_pkg]]
