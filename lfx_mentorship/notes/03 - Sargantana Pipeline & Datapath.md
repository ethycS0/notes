
The core inside the Sargantana tile (`top_drac` -> `datapath.sv`) implements an aggressive 64-bit RISC-V microarchitecture. It combines the simplicity and energy efficiency of **in-order instruction fetch and decode** with the throughput benefits of **register renaming, out-of-order multi-cycle execution, and in-order 2-wide graduation (ROB)**.

## 1. Architectural Summary

- **ISA Support**: RV64IMAFDC + Zicsr + Zifencei + Zba + Zbb + Zbs + Zicond + RISC-V Vector Extension (0.7.1 / 1.0 subsets).
- **Physical Register Files (PRF)**:
  - 64 Physical Integer Registers (vs 32 architectural `x` registers).
  - 64 Physical Floating-Point Registers (vs 32 architectural `f` registers).
  - 64 Physical Vector Registers (vs 32 architectural `v` registers).
- **Graduation List (ROB)**: 32 entries tracking in-flight speculative instructions.
- **Commit Width**: Up to **2 instructions retired per cycle**.
- **Speculative Checkpoints**: 4 hardware shadow checkpoints for zero-cycle rename table rollback on branch mispredictions.

## 2. Pipeline Stage Breakdown

```
 [IF1: Fetch 1]   --> PC Gen, Bimodal Predictor, RAS, iTLB & I-Cache Request
        |
 [IF2: Fetch 2]   --> Cacheline Reception, Alignment, Instruction Slicing
        |
 [ID:  Decode]    --> Instruction Decoder, Immediate Generator, Instruction Queue (16 entries)
        |
 [IR:  Rename]    --> Rename Table (RAT), Free List, Graduation List (ROB) Alloc, Checkpointing
        |
 [RR:  Reg Read]  --> Physical Regfile (PRF) Read, Scoreboards, Bypass / Forwarding Networks
        |
 [EXE: Execute]   --> ALU, Branch Unit, Mult, Radix-4 Div, LSU (LSQ/StoreBuf), FPU, Vector (VAGU)
        |
 [WB:  Writeback] --> Broadcast Results to PRF & Mark Ready in Graduation List
        |
 [Commit]         --> In-Order Retirement (2-wide), Free Old P-Regs, Drain Stores, Exception Trap
```

### Stage 1: Instruction Fetch 1 (`if_stage_1.sv`)

- **Primary Goal**: Generate the fetch PC for every cycle and initiate cache/TLB lookups.
- **Subunits**:
  - `branch_predictor.sv`: Bimodal 2-bit saturating counter branch prediction table.
  - `return_address_stack.sv` (RAS): Predicts subroutine return addresses (`ret` / `jalr`).
- **Next-PC Multiplexer**:
  Chooses the PC source among:
  1. Default sequential fetch: `PC + 4` (or `PC + 2` for compressed instructions).
  2. Branch Predictor / RAS target (predicted taken branches/jumps).
  3. Branch Redirect from Execution Unit (branch misprediction recovery).
  4. Trap / Exception Vector from CSR (`0x140` or `mtvec`).
  5. Debug Module entry or Replay.
- **Output**: Emits virtual/physical address to `nc_icache_buffer` and `sargantana_top_icache`.

### Stage 2: Instruction Fetch 2 (`if_stage_2.sv`)

- **Primary Goal**: Receive incoming cacheline beats, buffer them, and slice them into individual 32-bit instructions.
- **Operation**:
  - Receives 512-bit cachelines from L1 I-Cache (or 64-bit words from BootROM/uncacheable buffer).
  - Handles cacheline alignment and instruction packet extraction.
  - Passes the single aligned instruction, its corresponding PC, and prediction metadata to the Decode stage register (`stage_if_2_id_q`).

### Stage 3: Instruction Decode (`id_stage/`)

- **Primary Goal**: Parse the 32-bit instruction word into microarchitectural execution controls.
- **Subunits**:
  - `decoder.sv`: Decodes opcode, `funct3`, `funct7`, identifying target functional unit (ALU, Branch, Mult, Div, Mem, FPU, Vector, CSR), source registers (`rs1`, `rs2`, `rs3`), destination register (`rd`), and memory access sizing.
  - `immediate.sv`: Extracts and sign-extends immediate fields (I-, S-, B-, U-, J-type formats).
  - `vset_module.sv` & `vset_queue.sv`: Pre-decodes vector configuration instructions (`vsetvli`, `vsetivli`, `vsetvl`).
  - `instruction_queue.sv`: 16-entry FIFO buffer. Decouples the variable-latency frontend (fetch stalls/misses) from the backend execution engine.

### Stage 4: Instruction Rename / Dispatch (`ir_stage/`)

- **Primary Goal**: Eliminate false hazards (WAW, WAR) and assign speculative execution tracking resources.
- **Subunits**:
  - `rename_table.sv` (Register Alias Table - RAT): Maintains the current mapping between architectural registers (`x0-x31`, `f0-f31`, `v0-v31`) and physical registers (`p0-p63`).
  - `free_list.sv`: Tracks available, unallocated physical registers. Pops a fresh physical register (`prd`) for every instruction with a destination register.
  - `graduation_list.sv` (Allocation): Reserves a slot at the tail of the Reorder Buffer / Graduation List, recording instruction PC, assigned `prd`, and previous mapping `old_prd`.
  - **Speculative Checkpointing**: When a conditional branch is dispatched, a snapshot of the rename table is saved into one of 4 hardware checkpoint registers. If the branch later mispredicts, the rename table is instantly restored to that exact checkpoint in a single cycle.

### Stage 5: Register Read (`rr_stage/`)

- **Primary Goal**: Read source operand values and resolve data dependencies.
- **Subunits**:
  - `regfile.sv` (Physical Register File - PRF): Stores data for 64 physical registers (hardwired `p0 = 0`).
  - `score_board_scalar.sv` & `score_board_simd.sv`: Tracks the busy/ready status of every physical register.
  - **Bypass / Forwarding Multiplexers**: Intercepts operands in flight. If a required source register is being produced this cycle on a writeback bus, the value is forwarded directly into RR without waiting for PRF memory write.
  - **Hazard Stalling**: If a source operand is still being computed (e.g. outstanding L1 cache load or multi-cycle division), RR asserts `stall_rr` to hold execution.

### Stage 6: Execute (`exe_stage/`)

- **Primary Goal**: Perform computation, evaluate branch conditions, and access data memory.
- **Subunits**:
  - **ALU (`alu/`)**: Single-cycle arithmetic, logic, comparison, and bit manipulation instructions (Zba/Zbb/Zbs).
  - **Branch Unit (`branch_unit.sv`)**: Evaluates branch comparison conditions, calculates actual branch target addresses, compares with prediction metadata, and flags mispredictions to `control_unit.sv`.
  - **Multiplier (`mul_unit.sv`)**: Fully pipelined integer multiplier.
  - **Divider (`div_unit.sv`)**: Multi-cycle non-blocking Radix-4 divider (`div_4bits.sv`).
  - **Memory Unit / LSU (`mem_unit.sv`)**:
    - `load_store_queue.sv` (LSQ): Tracks in-flight memory requests.
    - `store_buffer.sv`: Holds speculative stores; drains to cache only after commit.
    - `pending_mem_req_queue.sv`: Manages outstanding cache requests.
    - Communicates with HPDCache over `dcache_interface`.
  - **FPU (`fpu/`)**: Single- and double-precision IEEE-754 floating-point pipelines.
  - **SIMD / Vector Unit (`simd/` & `vagu.sv`)**: Vector execution pipelines and Vector Address Generation Unit.
  - **CSR Unit Interface**: Routes CSR read/write commands to `csr_bsc.sv`.

### Stage 7: Writeback & Graduation / Commit (`wb_stage/`)

- **Writeback Phase**:
  - Execution units finish computation and drive writeback buses (`wb_scalar[0..3]`, `wb_fp[0..1]`, `wb_simd[0..1]`).
  - Results are written into the Physical Register File (`regfile.sv`).
  - Corresponding entries in the Graduation List are marked as **ready** (computed).
- **Graduation (Commit) Phase**:
  - The Graduation List retires ready instructions strictly in program order from its head pointer.
  - **2-Wide Superscalar Retirement**: Can retire up to **2 instructions per cycle**.
  - **Architectural Commit Actions**:
    1. The destination physical register `prd` becomes architecturally permanent.
    2. The previous physical register `old_prd` is pushed back into the `free_list.sv` to be reused.
    3. Speculative stores in the Store Buffer are authorized to write out to HPDCache.
    4. Retired instruction counts and PMU counters increment.
  - **Exception & Trap Handling**: If an instruction at the head of the Graduation List encountered an exception (e.g., page fault, illegal instruction), graduation halts, the entire pipeline and uncommitted Graduation List entries are purged, and the PC is redirected to the CSR trap handler.

## 3. Central Control Unit (`control_unit.sv`)

The Control Unit orchestrates all pipeline handshakes, hazard resolutions, and recovery events.

### Pipeline Stalls (`pipeline_ctrl_t`)

Stalls propagate backward from consumer stages to producer stages:
- `stall_wb`: Memory/structural backpressure stalling writeback.
- `stall_exe`: Stall during multi-cycle execution delays.
- `stall_rr`: Stall when source operands are not yet ready (RAW dependency).
- `stall_ir`: Stall when Free List is empty or Graduation List is full.
- `stall_id`: Stall when the Instruction Queue is full.
- `stall_if`: Stall when I-Cache is busy or experiencing a refill.

### Pipeline Flushes (`pipeline_flush_t`)

When a redirect occurs, the Control Unit clears invalidated instructions from pipeline registers:
- `flush_exe`: Kills mispredicted instructions currently in execution.
- `flush_rr`, `flush_ir`, `flush_id`, `flush_if`: Clears frontend and dispatch stages.
- Activates checkpoint rollback in `rename_table.sv` to restore the pre-branch architectural mapping in zero cycles.
