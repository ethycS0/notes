Decoder decodes the [[if_stage_2|Fetch2]] instruction in `decode_i` into the internal `instr_entry_t` structure `decode_instr_int`. It first copies the common instruction fields, extracts the immediate using [[immediate|Immediate COnstructor]], initializes all control fields to safe defaults, and then selects the instruction class from the opcode. The selected `func3`, `func7`, `func6`, or other subfields determine the exact `instr_type` and modify the operand, destination, functional unit, memory, FP, vector, and ordering fields.

The decoder also handles information that cannot be determined from the opcode alone. Privilege and CSR state are used for system and CMO instructions, FP CSR state is used for dynamic rounding, and vector state is supplied by [[vset_module|VSET Module]]. Branch instructions also update the RAS-related signals used by [[return_address_stack|Return Address Stack]]. Decode exceptions are generated after instruction decoding and preserve an exception already supplied by Fetch2.

The normal operation is therefore:

1. Copy instruction and pipeline information.
2. Generate the immediate.
3. Initialize decoded fields.
4. Decode the opcode and instruction subfields.
5. Set source and destination flags.
6. Select the execution unit.
7. Set instruction-specific metadata.
8. Check instruction legality.
9. Generate the final decoded instruction and exception information.

## Control Flags

The decoder initializes these flags to `0` and enables them for the instructions that require them.

| Flag               | Meaning                                                  | Example                        |
| ------------------ | -------------------------------------------------------- | ------------------------------ |
| `use_rs1`          | Instruction reads integer `rs1`                          | `ADD`, `LD`, `BEQ`             |
| `use_rs2`          | Instruction reads integer `rs2`                          | `ADD`, `SD`, `BEQ`             |
| `use_fs1`          | Instruction reads FP `fs1`                               | `FADD`, `VFADD.VF`             |
| `use_fs2`          | Instruction reads FP `fs2`                               | `FADD`                         |
| `use_fs3`          | Instruction reads FP `fs3`                               | `FMADD`                        |
| `use_vs1`          | Instruction reads vector `vs1`                           | `VADD.VV`                      |
| `use_vs2`          | Instruction reads vector `vs2`                           | `VADD.VV`                      |
| `use_mask`         | Instruction uses `v0` as the vector mask                 | Masked `VADD.VV`               |
| `use_old_vd`       | Instruction needs the previous value of `vd`             | `VMACC`                        |
| `use_imm`          | Operation explicitly consumes the decoded immediate      | `ADDI`, `LUI`, `JAL`           |
| `use_pc`           | Operation explicitly uses the instruction PC             | `AUIPC`, `JAL`                 |
| `regfile_we`       | Writes an integer register                               | `ADD`, `LD`                    |
| `vregfile_we`      | Writes a vector register                                 | `VADD`                         |
| `fregfile_we`      | Writes an FP register                                    | `FADD`, `FLW`                  |
| `is_opvx`          | Vector operation uses vector plus integer scalar form    | `OPIVX`                        |
| `is_opvi`          | Vector operation uses vector plus immediate form         | `OPIVI`                        |
| `is_opvf`          | Vector operation uses vector plus FP scalar form         | `OPFVF`                        |
| `op_32`            | Operation uses 32 bit integer semantics                  | `ADDW`, `MULW`                 |
| `signed_op`        | Execution should use signed arithmetic                   | `DIV`, `REM`                   |
| `stall_csr_fence`  | Instruction requires CSR, fence or vector fault ordering | `CSRRS`, `FENCE`               |
| `stall_vset_fence` | VSET instruction requires vector configuration ordering  | `VSETVL`, most `VSETVLI` cases |
| `ex_valid`         | Decoded instruction carries a decode exception           | Illegal instruction            |

use_mask is derived from vm (vm = 0 enables mask, vm = 1 disables it). use_old_vd defaults to 0 only when both tail and mask agnostic policies are active (vta & vma = 1), otherwise defaulting to 1. Individual instructions override use_old_vd: it is forced to 0 for whole-register transfers and mask operations (VL1R, VLM, VS1R, VSM), and forced to 1 for vector accumulators (VMADD, VMACC, etc.). Additionally, vregfile_we is masked to 0 whenever vl = 0.   

## Functional Unit

`unit` selects the execution path for the decoded instruction.

| Unit          | Used for                                                      |
| ------------- | ------------------------------------------------------------- |
| `UNIT_ALU`    | Integer ALU and default non-special operations                |
| `UNIT_MUL`    | Integer multiplication                                        |
| `UNIT_DIV`    | Integer division and remainder                                |
| `UNIT_MEM`    | Scalar memory, atomics, CMO and all vector memory operations  |
| `UNIT_BRANCH` | Branches and jumps                                            |
| `UNIT_SYSTEM` | CSR, privileged, system and VSET instructions                 |
| `UNIT_FPU`    | Scalar floating point                                         |
| `UNIT_SIMD`   | Vector arithmetic and vector operations                       |

The default unit is `UNIT_ALU`. An instruction only changes it when its execution path differs from the default. For example, `ADD` keeps `UNIT_ALU`, `MUL` selects `UNIT_MUL`, `LD` selects `UNIT_MEM`, and `VADD` selects `UNIT_SIMD`.

## Other Decode Fields

| Field        | Meaning                            | Example                           |
| ------------ | ---------------------------------- | --------------------------------- |
| `instr_type` | Exact operation selected by decode | `ADD`, `MUL`, `LD`, `VADD`        |
| `mem_type`   | Memory classification              | `LOAD`, `STORE`, `AMO`, `CMO_CBO` |
| `mem_size`   | Encoded memory access size         | `LB`, `LH`, `LW`, `LD`            |
| `signed_op`  | Signed arithmetic qualifier        | `DIV` versus `DIVU`               |
| `fmt`        | FP format                          | `FMT_S`, `FMT_D`, `FMT_H`         |
| `frm`        | FP rounding mode                   | `FADD`                            |
| `sew`        | Current vector element width       | Vector instructions               |
| `vl`         | Current vector length              | Vector instructions               |
| `lmul`       | Current vector LMUL                | Vector instructions               |
| `vta`        | Tail policy                        | Vector instructions               |
| `vma`        | Mask policy                        | Vector instructions               |
| `vlmax`      | Maximum vector length              | Vector instructions               |
| `vset_index` | Associated VSET queue entry        | Vector instructions               |

`mem_type` defaults to `NOT_MEM`. `instr_type` defaults to `ADD` and `unit` defaults to `UNIT_ALU`; the selected instruction case overwrites these values as necessary.

## Integer Instructions

| Instruction class      | Instructions                                                                                                                                                                                                                                                                                                                    |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Upper immediate        | `LUI`, `AUIPC`                                                                                                                                                                                                                                                                                                                  |
| Jumps                  | `JAL`, `JALR`                                                                                                                                                                                                                                                                                                                   |
| Branches               | `BEQ`, `BNE`, `BLT`, `BGE`, `BLTU`, `BGEU`                                                                                                                                                                                                                                                                                      |
| Loads                  | `LB`, `LH`, `LW`, `LD`, `LBU`, `LHU`, `LWU`                                                                                                                                                                                                                                                                                     |
| Stores                 | `SB`, `SH`, `SW`, `SD`                                                                                                                                                                                                                                                                                                          |
| Atomic 32 bit          | `AMO_LRW`, `AMO_SCW`, `AMO_SWAPW`, `AMO_ADDW`, `AMO_XORW`, `AMO_ANDW`, `AMO_ORW`, `AMO_MINW`, `AMO_MAXW`, `AMO_MINWU`, `AMO_MAXWU`                                                                                                                                                                                              |
| Atomic 64 bit          | `AMO_LRD`, `AMO_SCD`, `AMO_SWAPD`, `AMO_ADDD`, `AMO_XORD`, `AMO_ANDD`, `AMO_ORD`, `AMO_MIND`, `AMO_MAXD`, `AMO_MINDU`, `AMO_MAXDU`                                                                                                                                                                                              |
| Integer immediate      | `ADDI`, `SLTI`, `SLTIU`, `XORI`, `ORI`, `ANDI`, `SLLI`, `SRLI`, `SRAI`, `RORI`, `REV8`, `BSETI`, `BCLRI`, `BEXTI`, `BINVI`, `CPOP`, `CLZ`, `CTZ`, `SEXTB`, `SEXTH`, CMO prefetch operations                                                                                                                                     |
| Integer register       | `ADD`, `SUB`, `SLT`, `SLTU`, `XOR`, `OR`, `AND`, `SLL`, `SRL`, `SRA`, `ROL`, `ROR`, `XNOR`, `ORN`, `ANDN`, `MUL`, `MULH`, `MULHSU`, `MULHU`, `DIV`, `DIVU`, `REM`, `REMU`, `CLZ`, `CTZ`, `CPOP`, `BSET`, `BCLR`, `BEXT`, `BINV`, `REV8`, `ORCB`, `SH1ADD`, `SH2ADD`, `SH3ADD`, `MIN`, `MINU`, `MAX`, `MAXU`, `CZ_EQZ`, `CZ_NEZ` |
| Integer word immediate | `ADDIW`, `SLLIW`, `SRLIW`, `SRAIW`, `RORIW`, `CLZW`, `CTZW`, `CPOPW`                                                                                                                                                                                                                                                            |
| Integer word register  | `ADDW`, `SUBW`, `SLLW`, `SRLW`, `SRAW`, `RORW`, `ROLW`, `MULW`, `DIVW`, `DIVUW`, `REMW`, `REMUW`, `ADDUW`, `SH1ADDUW`, `SH2ADDUW`, `SH3ADDUW`, `SLLIUW`, `ZEXTH`                                                                                                                                                                |

## Floating Point Instructions

| Instruction class  | Instructions                                                        |
| ------------------ | ------------------------------------------------------------------- |
| Fused multiply add | `FMADD`, `FMSUB`, `FNMSUB`, `FNMADD`                                |
| Arithmetic         | `FADD`, `FSUB`, `FMUL`, `FDIV`, `FSQRT`                             |
| Sign manipulation  | `FSGNJ`                                                             |
| Min/max            | `FMIN_MAX`, `FMINM_MAXM`                                            |
| Conversion         | `FCVT_F2I`, `FCVTMOD`, `FCVT_I2F`, `FCVT_F2F`, `FROUND`, `FROUNDNX` |
| Compare            | `FCMP`, `FLEQ_FLTQ`                                                 |
| Move and classify  | `FMV_X2F`, `FMV_I2F`, `FCLASS`, `FLI`                               |

## Floating Point Memory

| Instruction class | Instructions        |
| ----------------- | ------------------- |
| FP loads          | `FLH`, `FLW`, `FLD` |
| FP stores         | `FSH`, `FSW`, `FSD` |

FP loads write the FP register file and use `UNIT_MEM`. FP arithmetic uses `UNIT_FPU`. FP to integer conversions switch the destination to the integer register file.

## Vector Memory Instructions

| Type                             | Instructions |
| -------------------------------- | ------------ |
| Unit stride load                 | `VLE`        |
| Unit stride whole register load  | `VL1R`       |
| Mask load                        | `VLM`        |
| Fault only first load            | `VLEFF`      |
| Strided load                     | `VLSE`       |
| Indexed load                     | `VLXE`       |
| Unit stride store                | `VSE`        |
| Unit stride whole register store | `VS1R`       |
| Mask store                       | `VSM`        |
| Strided store                    | `VSSE`       |
| Indexed store                    | `VSXE`       |

## Vector Integer Instructions

| Instruction group     | Instructions                                                                            |
| --------------------- | --------------------------------------------------------------------------------------- |
| Basic arithmetic      | `VADD`, `VSUB`, `VRSUB`                                                                 |
| Carry and borrow      | `VADC`, `VSBC`, `VMADC`, `VMSBC`                                                        |
| Min/max               | `VMIN`, `VMINU`, `VMAX`, `VMAXU`                                                        |
| Logic                 | `VAND`, `VOR`, `VXOR`, `VANDN`                                                          |
| Comparison            | `VMSEQ`, `VMSNE`, `VMSLTU`, `VMSLT`, `VMSLEU`, `VMSLE`, `VMSGTU`, `VMSGT`               |
| Saturating arithmetic | `VSADDU`, `VSADD`, `VSSUBU`, `VSSUB`                                                    |
| Shifts                | `VSLL`, `VSRL`, `VSRA`, `VSSRL`, `VSSRA`                                                |
| Narrowing             | `VNCLIPU`, `VNCLIP`, `VNSRL`, `VNSRA`                                                   |
| Gather and slide      | `VRGATHER`, `VRGATHEREI16`, `VSLIDEUP`, `VSLIDEDOWN`, `VSLIDE1UP`, `VSLIDE1DOWN`        |
| Move                  | `VMV1R`, `VMV_S_X`                                                                      |
| Multiply              | `VMULHU`, `VMUL`, `VMULHSU`, `VMULH`                                                    |
| Multiply add          | `VMADD`, `VNMSUB`, `VMACC`, `VNMSAC`                                                    |
| Widening multiply     | `VWMULU`, `VWMULSU`, `VWMUL`                                                            |
| Widening multiply add | `VWMACC`, `VWMACCU`, `VWMACCSU`, `VWMACCUS`                                             |
| Widening arithmetic   | `VWADDU`, `VWADD`, `VWSUBU`, `VWSUB`, `VWADDUW`, `VWADDW`, `VWSUBUW`, `VWSUBW`          |
| Widening shift        | `VWSLL`                                                                                 |
| Reduction             | `VREDSUM`, `VREDAND`, `VREDOR`, `VREDXOR`, `VREDMINU`, `VREDMIN`, `VREDMAXU`, `VREDMAX` |
| Mask reduction        | `VMAND`, `VMNAND`, `VMANDN`, `VMNOR`, `VMORN`, `VMOR`, `VMXOR`, `VMXNOR`                |
| Mask and index        | `VCNT`, `VID`, `VIOTA`, `VMSBF`, `VMSIF`, `VMSOF`                                       |
| Vector to scalar      | `VMV_X_S`, `VPOPC`, `VFIRST`                                                            |
| Extension             | `VZEXT_VF8`, `VSEXT_VF8`, `VZEXT_VF4`, `VSEXT_VF4`, `VZEXT_VF2`, `VSEXT_VF2`            |
| Bit manipulation      | `VBREV8`, `VREV8`, `VBREV`, `VCLZ`, `VCTZ`, `VCPOP`                                     |
| Compression           | `VCOMPRESS`                                                                             |
| Average and subtract  | `VAADDU`, `VAADD`, `VASUBU`, `VASUB`                                                    |
| Divide and remainder  | `VDIVU`, `VDIV`, `VREMU`, `VREM`                                                        |

## Vector Floating Point Instructions

| Instruction group           | Instructions                                                                                                                                                                                                                                                                                                                        |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Arithmetic                  | `VFADD`, `VFSUB`, `VFMUL`, `VFDIV`, `VFRSUB`, `VFRDIV`                                                                                                                                                                                                                                                                              |
| Fused multiply add          | `VFMADD`, `VFMSUB`, `VFMACC`, `VFMSAC`, `VFNMADD`, `VFNMSUB`, `VFNMACC`, `VFNMSAC`                                                                                                                                                                                                                                                  |
| Min/max                     | `VFMIN`, `VFMAX`                                                                                                                                                                                                                                                                                                                    |
| Sign manipulation           | `VFSGNJ`, `VFSGNJN`, `VFSGNJX`                                                                                                                                                                                                                                                                                                      |
| Comparison                  | `VMFEQ`, `VMFLE`, `VMFLT`, `VMFNE`, `VMFGT`, `VMFGE`                                                                                                                                                                                                                                                                                |
| Widening arithmetic         | `VFWADD`, `VFWSUB`, `VFWMUL`, `VFWADDW`, `VFWSUBW`                                                                                                                                                                                                                                                                                  |
| Widening fused multiply add | `VFWMACC`, `VFWNMACC`, `VFWMSAC`, `VFWNMSAC`                                                                                                                                                                                                                                                                                        |
| Conversion                  | `VFNCVT_F_F`, `VFNCVT_F_X`, `VFNCVT_F_XU`, `VFNCVT_X_F`, `VFNCVT_RTZ_X_F`, `VFNCVT_RTZ_XU_F`, `VFNCVT_XU_F`, `VFNCVT_ROD_F_F`, `VFWCVT_RTZ_X_F`, `VFWCVT_F_F`, `VFWCVT_F_X`, `VFWCVT_F_XU`, `VFWCVT_X_F`, `VFWCVT_XU_F`, `VFWCVT_RTZ_XU_F`, `VFCVT_XU_F`, `VFCVT_X_F`, `VFCVT_F_XU`, `VFCVT_F_X`, `VFCVT_RTZ_XU_F`, `VFCVT_RTZ_X_F` |
| Unary                       | `VFCLASS`, `VFSQRT`, `VFRSQRT7`, `VFREC7`                                                                                                                                                                                                                                                                                           |
| Reduction                   | `VFREDUSUM`, `VFREDMAX`, `VFREDMIN`, `VFREDOSUM`, `VFWREDUSUM`, `VFWREDOSUM`                                                                                                                                                                                                                                                        |
| Move                        | `VFMV`, `VFMV_F_S`, `VFMV_S_F`                                                                                                                                                                                                                                                                                                      |
| Merge                       | `VFMERGE`                                                                                                                                                                                                                                                                                                                           |
| Slide                       | `VFSLIDE1UP`, `VFSLIDE1DOWN`                                                                                                                                                                                                                                                                                                        |

## Vector Configuration

| Instruction | `unit`        | Main controls                                        |
| ----------- | ------------- | ---------------------------------------------------- |
| `VSETVL`    | `UNIT_SYSTEM` | `use_rs1`, `use_rs2`, `stall_vset_fence`             |
| `VSETVLI`   | `UNIT_SYSTEM` | `use_rs1`, `use_imm`, conditional `stall_vset_fence` |
| `VSETIVLI`  | `UNIT_SYSTEM` | `use_imm`                                            |

The vector configuration is maintained by [[vset_module|VSET Module]]. Its outputs include `vl`, `sew`, `lmul`, `vta`, `vma`, `vlmax`, `vill`, and the VSET queue index. These values are included in the decoded vector instruction metadata.

## Fence and CMO

| Instruction | `instr_type` | `unit`             | `stall_csr_fence` |
| ----------- | ------------ | ------------------ | ----------------: |
| `FENCE`     | `FENCE`      | Default `UNIT_ALU` |               `1` |
| `FENCE.I`   | `FENCE_I`    | Default `UNIT_ALU` |               `1` |
| `CBO.INVAL` | `CBO_INVAL`  | `UNIT_MEM`         |               `0` |
| `CBO.CLEAN` | `CBO_CLEAN`  | `UNIT_MEM`         |               `0` |
| `CBO.FLUSH` | `CBO_FLUSH`  | `UNIT_MEM`         |               `0` |
| `CBO.ZERO`  | `CBO_ZERO`   | `UNIT_MEM`         |               `0` |

The CMO instructions perform privilege and CSR permission checks before being accepted.

## System Instructions

| Instruction group          | Instructions                                                                                                               |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Hypervisor load/store      | `HLV_B`, `HLV_BU`, `HSV_B`, `HLV_H`, `HLV_HU`, `HLVX_HU`, `HSV_H`, `HLV_W`, `HLV_WU`, `HLVX_WU`, `HSV_W`, `HLV_D`, `HSV_D` |
| Trap and return            | `ECALL`, `EBREAK`, `URET`, `SRET`, `MRET`, `HRTS`                                                                          |
| Wait                       | `WFI`                                                                                                                      |
| Address translation fences | `SFENCE_VMA`, `HFENCE_VVMA`, `HFENCE_GVMA`                                                                                 |
| CSR register               | `CSRRW`, `CSRRS`, `CSRRC`                                                                                                  |
| CSR immediate              | `CSRRWI`, `CSRRSI`, `CSRRCI`                                                                                               |

System instructions use `UNIT_SYSTEM` unless the implementation explicitly routes the operation elsewhere, such as hypervisor load/store operations using the memory unit.

## Exceptions

Decode internally tracks:

| Signal                         | Meaning                                          |
| ------------------------------ | ------------------------------------------------ |
| `xcpt_addr_misaligned_int`     | Instruction target is misaligned                 |
| `xcpt_illegal_c_instr_int`     | Compressed encoding is unsupported               |
| `xcpt_illegal_instruction_int` | Instruction encoding or operands are illegal     |
| `xcpt_virtual_instruction_int` | Instruction is illegal because it is virtualized |

If `decode_i.ex.valid` is already set, the decoder forwards that exception unchanged. Otherwise it generates the appropriate decoder exception. The resulting exception is stored in `decode_instr_o.ex` and `decode_instr_o.instr.ex_valid` is set accordingly.

## Special Interfaces

`jal_id_if_o` carries early jump information for JAL and RAS based JALR handling. The decoder also generates `ras_push_int` and `ras_pop_int`, which are consumed by [[return_address_stack|Return Address Stack]].

`full_vset_queue_o`, `prev_vtype_o`, and `vl_short_o` are supplied by the VSET machinery.

The decoder therefore acts as the boundary between the raw instruction encoding and the richer internal representation required by the rest of the Sargantana pipeline.
