# immediate

File: `rtl/datapath/rtl/id_stage/rtl/immediate.sv` (`module immediate`, line 21)
Instantiated in [[decoder]] `decoder.sv:98`.

## Overview

Pure combinational immediate generator. Reconstructs and sign-extends the
immediate for every RISC-V instruction format, including vector variants.

## Ports (immediate.sv:24–27)

`instr_i` (`instruction_t`) → `imm_o` (`bus64_t`).

## Behaviour

- I/S/B/U/J fields `:43`–`:54`.
- Vector immediates `:56`–`:60`.
- Format select `case (instr_i.common.opcode)` `:63`–`:173`, covering LUI/AUIPC,
  JAL, LOAD-FP, JALR/LOAD, ALU-I, ALU-I-W, BRANCH, FP, STORE-FP, STORE, V,
  SYSTEM.

## Related

[[immediate]] · [[decoder]] · [[riscv_pkg]]
