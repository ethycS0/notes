# regfile

File: `rtl/datapath/rtl/rr_stage/rtl/regfile.sv` (`module regfile`, line 23)
Instantiated in [[datapath]] `datapath.sv:1015` (scalar), `:1035` (FP),
`:1057` (vector).

## Overview

Generic multi-read / multi-write register file with same-cycle write-through
bypass and optional hardwired zero. Parameterised by `NUM_READ_PORTS`,
`NUM_WRITEBACK_PORTS`, `NUM_REGISTERS`, `HARDWIRED_ZERO`, `reg_type`,
`data_type`.

## Ports (regfile.sv:34–46)

`clk_i`, `rstn_i`, `write_enable_i`, `write_addr_i`, `write_data_i`,
`read_addr_i`; output `read_data_o`.

## Behaviour

- `LOWEST_REGISTER = HARDWIRED_ZERO ? 1 : 0` `:49`.
- Bypass matrix `:57` (write port to read port same-cycle).
- Read mux: hardwired zero → bypass → array `:71`.
- Synchronous write `:84`.

## Usage

- Scalar: `NUM_READ_PORTS=2`, `NUM_WRITEBACK_PORTS=4`, 64 regs.
- FP: `NUM_READ_PORTS=3`, `NUM_WRITEBACK_PORTS=2`, 64 regs.
- Vector: `NUM_READ_PORTS=4`, `NUM_WRITEBACK_PORTS=2`, 64 regs.

## Related

[[regfile]] · [[rr_stage]] · [[rename_table]] · [[drac_pkg]]
