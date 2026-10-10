# RR Stage (Read Registers)

Files:

- `rtl/datapath/rtl/rr_stage/rtl/regfile.sv`
- `rtl/datapath/rtl/wb_stage/rtl/graduation_list.sv` (allocation happens here)

Wired in [[datapath]]: regfiles `datapath.sv:1015` (scalar), `:1035` (FP),
`:1057` (vector); [[graduation_list]] `datapath.sv:948`.

## Purpose

Read the physical register operands addressed by [[rename_table]] and allocate an
in-order slot in the [[graduation_list]] for the instruction.

## Regfile

[[regfile]] is a generic multi-port register file with write-through bypass
(`regfile.sv:57`). Instances:

- Scalar: 2 read / 4 write ports, 64×64-bit, hardwired zero (`datapath.sv:1015`).
- FP: 3 read / 2 write, 64×64-bit (`datapath.sv:1035`).
- Vector: 4 read / 2 write, 64×VLEN (`datapath.sv:1057`).

FP/scalar operand mux based on `use_fs1/use_fs2` (`datapath.sv:1080`).

## Graduation list allocation

`assigned_gl_entry_o` (`datapath.sv:971`) becomes `stage_rr_exe_d.gl_index`.
GL depth = 32 entries (`graduation_list.sv:25`); `gl_full` back-pressures RR
(`rr_cu_int.gl_full`, `datapath.sv:974`).

## Snooping

WB results in flight are snooped into the ready bits so dependent instructions
issue without a full round trip (`datapath.sv:982`–`:1010`).

## RR→EXE register

`reg_rr_inst` (`datapath.sv:1128`) registers `rr_exe_instr_t` (see [[drac_pkg]]
`drac_pkg.sv:594`); a bypass mux (`datapath.sv:1125`) re-uses `reg_to_exe` when
stalled.

## Related

[[rr_stage]] · [[regfile]] · [[graduation_list]] · [[drac_pkg]]
