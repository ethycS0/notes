# WB Stage (Write-Back)

Files:

- `rtl/datapath/rtl/wb_stage/rtl/graduation_list.sv`
- Write-back logic lives in [[datapath]] `datapath.sv:1373`–`:1520`.

## Purpose

Collect unit results, update the [[graduation_list]] entry of each instruction
(marking it finished), drive register-file write ports, and expose bypass data
to [[exe_stage]] / [[rr_stage]].

## Write-back assembly (`datapath.sv:1381`–`:1484`)

- Scalar (`NUM_SCALAR_WB = 4`): `gl_valid`/`gl_index`, `data_wb_to_exe`,
  `write_paddr_exe`. Port 1 masks AMO (`datapath.sv:1378`, `:1385`).
- SIMD (`NUM_SIMD_WB = 2`) and FP (`NUM_FP_WB = 2`) analogous.
- `wb_cu_int` ([[drac_pkg]] `wb_cu_t`) reports `valid`, `write_enable`,
  `snoop_enable`, `checkpoint_done`, `chkp`, `gl_index`.

## Data to regfile (`datapath.sv:1488`–`:1520`)

- Port 0 carries either debug write, CSR read data, VSETVL result, or the normal
  result (`datapath.sv:1496`).
- `write_paddr_rr` selects commit physical dest or WB physical dest (`:1503`).

## Graduation-list update

The `instruction_writeback_*` arrays feed [[graduation_list]] at
`graduation_list.sv:137`–`:185`; a slot becomes valid once its result arrives.

## Related

[[wb_stage]] · [[graduation_list]] · [[drac_pkg]]
