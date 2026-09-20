
Immediate Constructor extracts the immediate or immediate like value from a decoded RISC V instruction encoding and produces a single 64 bit `imm_o` output.

The module handles standard RISC V immediate formats, shift amounts, vector immediates, vector configuration fields, and selected FP and SYSTEM instruction fields. The decoder determines whether the value is actually used by the instruction. 

## Internal Immediate Values

| Signal          | Width | Description                                                 |
| --------------- | ----: | ----------------------------------------------------------- |
| `imm_itype`     |    32 | Sign extended I type immediate                              |
| `imm_stype`     |    32 | Sign extended S type immediate                              |
| `imm_btype`     |    32 | Sign extended B type immediate                              |
| `imm_utype`     |    32 | U type immediate with low 12 bits cleared                   |
| `imm_jtype`     |    32 | Sign extended J type immediate                              |
| `imm_uitype`    |    64 | Zero extended `rs1` field, currently unused                 |
| `imm_vtype`     |    64 | Sign extended vector `vs1` field                            |
| `imm_uvtype`    |    64 | Zero extended vector `vs1` field                            |
| `imm_uvrtype`   |    64 | Special 6 bit value for `VROR.vi`                           |
| `imm_shamt`     |    64 | Shift amount from `instr[25:20]`                            |
| `imm_shamt_big` |    64 | Shift field from `instr[26:20]`                             |
| `sign_extended` |    32 | Replicated `instr[31]` used for 32 to 64 bit sign extension |

## Standard Immediate Construction

The module constructs the standard RISC V formats before selecting one of them.

| Format | Source fields                                            | Construction                             |
| ------ | -------------------------------------------------------- | ---------------------------------------- |
| I type | `instr[31:20]`                                           | Sign extended to 32 bits                 |
| S type | `instr[31:25]`, `instr[11:7]`                            | Reconstruct and sign extend              |
| B type | `instr[31]`, `instr[30:25]`, `instr[11:8]`, `instr[7]`   | Reconstruct branch offset and append `0` |
| U type | `instr[31:12]`                                           | Append 12 zero bits                      |
| J type | `instr[31]`, `instr[30:21]`, `instr[20]`, `instr[19:12]` | Reconstruct jump offset and append `0`   |

`sign_extended` is later concatenated with 32 bit values to form the final 64 bit signed value.


## Shift Amounts

| Signal          | Source                                   | Used by                   |
| --------------- | ---------------------------------------- | ------------------------- |
| `imm_shamt_big` | `instr[26:20]` with `instr[26]` extended | RV64 ALU immediate shifts |
| `imm_shamt`     | `instr[25:20]`                           | RV64 W shifts             |

`OP_ALU_I` selects `imm_shamt_big` for `SLLI` and the combined `SRLI` or `SRAI` encoding represented by `F3_SRLAI`.

`OP_ALU_I_W` selects `imm_shamt` for the W shift encodings represented by `F3_64_SLLIW` and `F3_64_SRLIW_SRAIW`.

For valid instructions, the encoding constraints determine the additional upper shift encoding bit.

## Vector Immediate Values

| Signal        | Source                 | Extension   | Purpose                                      |
| ------------- | ---------------------- | ----------- | -------------------------------------------- |
| `imm_vtype`   | `instr_i.vtype.vs1`    | Sign extend | General vector immediate                     |
| `imm_uvtype`  | `instr_i.vtype.vs1`    | Zero extend | Vector instructions requiring unsigned value |
| `imm_uvrtype` | `instr[26]` plus `vs1` | Zero extend | Special `VROR.vi` immediate                  |

The same physical `vs1` field is therefore interpreted differently depending on the vector operation.

## Opcode Selection

| Opcode        | Secondary selection   | `imm_o`                                     |
| ------------- | --------------------- | ------------------------------------------- |
| `OP_LUI`      | None                  | U type, sign extended to 64 bits            |
| `OP_AUIPC`    | None                  | U type, sign extended to 64 bits            |
| `OP_JAL`      | None                  | J type, sign extended to 64 bits            |
| `OP_LOAD_FP`  | `func3`               | I type for `FLH`, `FLW`, `FLD`              |
| `OP_JALR`     | None                  | I type                                      |
| `OP_LOAD`     | None                  | I type                                      |
| `OP_ALU_I`    | `func3`               | Shift amount for shifts, otherwise I type   |
| `OP_ALU_I_W`  | `func3`               | W shift amount for shifts, otherwise I type |
| `OP_BRANCH`   | None                  | B type                                      |
| `OP_FP`       | None                  | Zero extended `instr[19:15]`                |
| `OP_STORE_FP` | `func3`               | S type for `FLH`, `FLW`, `FLD`              |
| `OP_STORE`    | None                  | S type                                      |
| `OP_V`        | `func3`, then `func6` | Vector specific value                       |
| `OP_SYSTEM`   | `func3`               | I type field for selected SYSTEM operations |
| Other         | None                  | `0`                                         |

## ALU Immediate Operations

For `OP_ALU_I`, the module checks `func3`.

| Operation encoding          | Immediate source |
| --------------------------- | ---------------- |
| `F3_SLLI`                   | `imm_shamt_big`  |
| `F3_SRLAI`                  | `imm_shamt_big`  |
| Other `OP_ALU_I` operations | I type           |

For `OP_ALU_I_W`:

| Operation encoding            | Immediate source |
| ----------------------------- | ---------------- |
| `F3_64_SLLIW`                 | `imm_shamt`      |
| `F3_64_SRLIW_SRAIW`           | `imm_shamt`      |
| Other `OP_ALU_I_W` operations | I type           |

This keeps normal ALU immediates and shift amounts on the same `imm_o` interface while changing only the selected source.

## Floating Point Operations

### FP Loads

`OP_LOAD_FP` uses the I type immediate only when `func3` is `F3_FLH`, `F3_FLW`, or `F3_FLD`.

Other encodings under this opcode produce zero.

### FP Stores

`OP_STORE_FP` uses the S type immediate for `F3_FLH`, `F3_FLW`, and `F3_FLD`.

Other encodings produce zero.

### `OP_FP`

`OP_FP` directly places `instr[19:15]` into `imm_o` with zero extension:

```systemverilog
imm_o = {{28{1'b0}}, instr_i.bits[19:15]};
```

The source does not restrict this extraction to one specific FP instruction. The semantic interpretation of this field is handled elsewhere.

## Vector Operations

`OP_V` first checks `func3`.

### `F3_OPIVI`

The `func6` field determines which vector immediate representation is selected.

| `func6` group            | Immediate     |
| ------------------------ | ------------- |
| `F6_VSLL`                | `imm_uvtype`  |
| `F6_VSRL`                | `imm_uvtype`  |
| `F6_VSRA`                | `imm_uvtype`  |
| `F6_VNSRL`               | `imm_uvtype`  |
| `F6_VNSRA`               | `imm_uvtype`  |
| `F6_VSLIDEUP`            | `imm_uvtype`  |
| `F6_VSLIDEDOWN`          | `imm_uvtype`  |
| `F6_VRGATHER`            | `imm_uvtype`  |
| `F6_VNCLIPU`             | `imm_uvtype`  |
| `F6_VNCLIP`              | `imm_uvtype`  |
| `F6_VROR`                | `imm_uvrtype` |
| Other `OPIVI` operations | `imm_vtype`   |

This means vector immediate instructions do not all interpret the 5 bit `vs1` field in the same way.

### `F3_OPCFG`

For vector configuration instructions, the module checks:

```systemverilog
instr_i[31] & instr_i[30]
```

When true, it selects the `VSETIVLI` encoding and builds:

```text
27 zero bits
5 bits from instr[19:15]
22 zero bits
10 bits from instr[29:20]
```

into `imm_o`.

Otherwise, `imm_itype` is selected.

The `VSETIVLI` value is therefore an implementation specific packed configuration value rather than a normal arithmetic immediate.

### Other Vector Operations

For other `OP_V` `func3` values, `imm_vtype` is selected.

## SYSTEM Operations

`OP_SYSTEM` checks `instr_i.itype.func3`.

| Encoding               | `imm_o`      |
| ---------------------- | ------------ |
| `F3_CSRRW`             | I type field |
| `F3_CSRRS`             | I type field |
| `F3_CSRRC`             | I type field |
| `F3_CSRRWI`            | I type field |
| `F3_CSRRSI`            | I type field |
| `F3_CSRRCI`            | I type field |
| `F3_ECALL_EBREAK_ERET` | I type field |
| Other                  | `0`          |

The complete I type field is passed through `imm_o`. For CSR instructions, this includes the CSR address field in `instr[31:20]`. The separate `instr[19:15]` field is not replaced by `imm_o`.

## Output Behavior

`imm_o` is selected entirely from the instruction opcode and, where required, `func3` or `func6`.

The default case is:

```systemverilog
imm_o = 64'b0;
```

The same zero result is used for unsupported subencodings of FP loads, FP stores, and SYSTEM instructions.

A zero value here does not indicate whether the instruction uses an immediate. Immediate consumption is determined by the decode logic in [[decoder.sv]].

## Data Flow

```text
instr_i
   |
   +-- I S B U J reconstruction
   +-- shift amount extraction
   +-- vector immediate extraction
   +-- FP field extraction
   +-- SYSTEM field extraction
   |
   v
opcode / func3 / func6 selection
   |
   v
imm_o
```

The module is therefore an immediate extraction and formatting stage. It normalizes different instruction encodings into one 64 bit value so the rest of the decode pipeline does not need to reconstruct each format again.
