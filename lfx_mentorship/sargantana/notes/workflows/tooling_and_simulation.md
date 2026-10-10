# Tooling & Simulation Workflow

The **Sargantana `core_tile`** verification and simulation ecosystem is built to give full visibility into hardware execution without needing to manually probe every net in RTL from scratch.

This guide details how to build, run, trace, and debug the tile using **Verilator**, **Commit Logs**, **Konata Pipeline Diagrams**, and **Waveform Traces**.

> Paths in this document are relative to the **repository root**, not to
> `core_tile/rtl/core/sargantana/`. For datapath RTL details see [[pipeline_overview]].

## 1. Simulator Architecture

The primary simulation environment is driven by **Verilator** (compiling SystemVerilog to high-speed multithreaded C++ binaries) with SystemVerilog Direct Programming Interface (**DPI-C**) extensions for ELF loading, instruction disassembly, and trace dumping.

```
+--------------------------------------------------------------------------+
|                            sim_top (Testbench)                           |
|                                                                          |
|  +--------------------+   +-----------------------+   +---------------+  |
|  | BootROM (0x100)    |   | L2 Memory / Host DPI  |   | SimJTAG / DM  |  |
|  +---------+----------+   +-----------+-----------+   +-------+-------+  |
|            |                          |                       |          |
|  +---------v--------------------------v-----------------------v-------+  |
|  |                              top_tile (DUT)                        |  |
|  |  +---------------------------------------------------------------+ |  |
|  |  |  sargantana_subtile                                           | |  |
|  |  |  +---------------------------------------------------------+  | |  |
|  |  |  |  top_drac -> datapath.sv                                |  | |  |
|  |  |  |  (DPI Instrumentation: Commit Log, Konata Dump)         |  | |  |
|  |  |  +---------------------------------------------------------+  | |  |
|  |  +---------------------------------------------------------------+ |  |
|  +--------------------------------------------------------------------+  |
+--------------------------------------------------------------------------+
```

### Core C++ / DPI Modules (`simulator/models/cxx/`)

1. **`loadelf.cpp`**: Parses ELF binaries and directly pre-populates target memory before execution begins.
2. **`dpi_commit_log.cpp`**: Monitors the graduation/commit stage, decodes instructions via `libdisasm` (from Spike), and writes dynamic retirement records to `signature.txt`.
3. **`dpi_konata.cpp`**: Hooks cycle-by-cycle stage transitions (`if1`, `if2`, `id`, `ir`, `rr`, `exe`, `wb`) and exports a visual timeline format for the Konata viewer.
4. **`dpi_perfect_memory.cpp` / `l2_behav.sv`**: Emulates the L2 cache and main memory subsystem, servicing L1 instruction cache fills and HPDCache writebacks/misses.
5. **`dpi_checkpoint.cpp`**: Implements snapshot and state restoration (`--savable`) for fast-forwarding long benchmark runs.

## 2. Compilation & Build

From the root of the repository

```bash
# Build bootrom, libdisasm, and Verilator simulator binary (./sim)
make -j$(nproc) sim

# Optional: Build ISA test suite
make -j$(nproc) build-isa-tests

# Optional: Build benchmarks suite
make -j$(nproc) build-benchmarks
```

The resulting simulation executable is located directly in the root as `./sim`.

## 3. Simulator Runtime Options (Plusargs)

`./sim` accepts runtime options via SystemVerilog plusargs (`+<arg>` or `+<arg>=<value>`):

| Plusarg                        | Default Value           | Description                                                  |
| :----------------------------- | :---------------------- | :----------------------------------------------------------- |
| `+load=<path>`                 | _(Mandatory)_           | Path to RISC-V ELF binary to execute.                        |
| `+commit_log[=<file>]`         | `signature.txt`         | Enables instruction commit logging to file.                  |
| `+konata_dump[=<file>]`        | `konata.txt`            | Enables pipeline stage event dumping for Konata.             |
| `+vcd[=<file>]`                | `dump_file.vcd`         | Enables FST/VCD waveform dumping.                            |
| `+start-vcd-cycles=<N>`        | `0`                     | Delays waveform dumping until simulation cycle `N`.          |
| `+max-cycles=<N>`              | `0` (unlimited)         | Maximum execution cycles before asserting timeout error.     |
| `+deadlock-cycles=<N>`         | `200`                   | Fires an assertion if no instruction commits for `N` cycles. |
| `+checkpoint_Mcycles=<N>`      | `0` (disabled)          | Saves Verilator model snapshot every `N` million cycles.     |
| `+checkpoint_name=<prefix>`    | `verilator_model`       | Base file prefix for saving snapshots (`_1.bin`, `_2.bin`).  |
| `+checkpoint_restore_ON`       | Disabled                | Resumes execution from an existing snapshot file.            |
| `+checkpoint_restore_name=<f>` | `verilator_model_1.bin` | Snapshot file to restore from.                               |
| `+jtag`                        | Disabled                | Enables OpenOCD / JTAG remote simulation connection.         |

> [!TIP]
> **Performance Tip**: Running with both `+vcd` and full logs slows down simulation significantly. For fast profiling, run only with `+commit_log` or `+konata_dump`. When hunting low-level gate/hazard bugs, identify the cycle of divergence first, then rerun with `+vcd +start-vcd-cycles=<target_cycle - 200>`.

## 4. Deep Dive: Commit Log (`signature.txt`)

The Commit Log is the fastest way to verify architectural correctness. Every time an instruction reaches the head of the [[graduation_list]] and legitimately commits, `dpi_commit_log.cpp` records the architectural state mutation.

### Log Output Format

Each retired instruction emits two lines:

```text
core   0: 0x0000000000000100 (0x0010041b) addiw   s0, zero, 1
core    0:  3 0x0000000000000100 (0x0010041b) x8  0x0000000000000001
```

1. **Line 1: Disassembly & PC**
   - `core <id>`: Hart index (hart 0).
   - `0x0000000000000100`: Virtual/Physical Program Counter (PC).
   - `(0x0010041b)`: 32-bit machine instruction word.
   - `addiw s0, zero, 1`: Disassembled assembly mnemonic and operands.

2. **Line 2: Register Writeback & Privilege Level**
   - `<priv>`: Privilege level (`3` = Machine, `1` = Supervisor, `0` = User).
   - `<dst_reg>`: Destination register (e.g. `x8` for integer, `f10` for FP).
   - `<data>`: Final 64-bit value written back into the register.

### Memory & Special Instructions

- **Memory Operations**:
  ```text
  core    0:  3 0x0000000080000150 (0x00043503) ld   a0, 0(s0) mem 0x0000000080002000
  ```
  Appends `mem <address>` indicating the effective memory address computed and accessed.
- **Trap / Exception**:
  ```text
  core   0: exception trap_illegal_instruction, epc 0x0000000080000000
  core   0:           tval 0x0000000000000000
  ```
  Immediately flags unexpected faults, giving the exact `epc` and `tval`.

### Practical Use Case: Locating Execution Divergence

If your core behavior differs from expected or fails a test:

1. Run with `+commit_log=my_sig.txt`.
2. Inspect the last committed instructions before failure:
   ```bash
   tail -n 40 my_sig.txt
   ```
3. Look for:
   - Infinite loops (`j` / `bne` jumping to itself).
   - Traps to `0x140` (the default trap vector in BootROM).
   - A register value that diverges from expected output.

## 5. Deep Dive: Konata Pipeline Visualizer (`konata.txt`)

Konata displays a graphical instruction pipeline diagram where time flows horizontally (cycles) and instructions flow vertically.

### How Konata Traces Are Structured

`dpi_konata.cpp` logs the following primitives into `konata.txt`:

```text
C    1                # Advance simulation by 1 cycle
I    4    4    0      # Introduce instruction ID 4
S    4    0    F1     # Instruction 4 enters stage F1 (Fetch 1)
E    4    0    F1     # Instruction 4 exits stage F1
S    4    0    F2     # Instruction 4 enters stage F2 (Fetch 2)
L    4    0    0x80000000: addi a0, a0, 1 # Label with disassembly
S    4    0    D      # Stage D (Decode)
S    4    0    Q      # Stage Q (Instruction Queue)
S    4    0    I      # Stage I (Rename & Issue)
S    4    0    R      # Stage R (Register Read)
S    4    0    A      # Stage A (ALU Execution)
R    4    4    0      # Retire instruction 4 (0 = committed successfully)
R    5    5    1      # Retire instruction 5 (1 = squashed/flushed!)
```

### Stage Abbreviations in Sargantana

| Label     | Pipeline Stage     | Meaning & Diagnostic Value                                                |
| :-------- | :----------------- | :------------------------------------------------------------------------ |
| **`F1`**  | Fetch Stage 1      | PC selection and I-Cache/TLB tag request.                                 |
| **`F2`**  | Fetch Stage 2      | Receiving cacheline and instruction alignment.                            |
| **`D`**   | Instruction Decode | Decoding opcode, immediate extraction.                                    |
| **`Q`**   | Instruction Queue  | Buffers decoded instructions awaiting rename. Long bar = backend stalled! |
| **`I`**   | Rename / Dispatch  | Renames arch regs to physical regs, allocates ROB entry.                  |
| **`R`**   | Register Read      | Accessing PRF and bypass networks.                                        |
| **`A`**   | ALU Unit           | Execution in Integer ALU (single cycle).                                  |
| **`DIV`** | Divider Unit       | Multi-cycle iterative division.                                           |
| **`MUL`** | Multiplier Unit    | Pipelined multiplication.                                                 |
| **`B`**   | Branch Unit        | Branch condition evaluation and target check.                             |
| **`M`**   | Memory / LSU       | Load/Store Queue, D-Cache access. Long bar = cache miss or L2 refill.     |
| **`V`**   | Vector Unit        | Vector arithmetic and VAGU memory operations.                             |
| **`FP`**  | FPU Unit           | Floating-point pipeline.                                                  |
| **`E`**   | System / CSR       | CSR reads, writes, and privilege switches.                                |

### Visual Patterns to Identify in Konata

1. **Pipeline Bubble**: Horizontal gap between `F1` and `D` indicates I-Cache misses or branch redirect delays.
2. **Structural Stall**: Instruction stays stuck in `Q` or `I` because the [[free_list]] is empty (no free physical registers) or [[graduation_list]] (ROB) is full.
3. **Data Dependency Stall**: Instruction reaches `R` (Register Read) and waits several cycles because its source register is being computed by an in-flight `M` (Load) or `DIV`.
4. **Branch Misprediction Flush**: When a branch resolves in `B`, all subsequent younger instructions turn red with `R <id> <id> 1` (flushed), and new instructions enter `F1` at the corrected target PC.

## 6. Deep Dive: Waveform Debugging (GTKWave / Surfer)

Waveforms give bit-exact insight into every wire, clock edge, and register state.

### Generating Waveforms

```bash
# Generate waveform for an entire run (can become several gigabytes for long runs):
./sim +load=benchmarks/benchmarks/custom_tests.riscv +vcd=dump_file.vcd

# Target a specific cycle window (e.g. from cycle 1200):
./sim +load=benchmarks/benchmarks/custom_tests.riscv +vcd=dump_file.vcd +start-vcd-cycles=1200
```

### Opening Waveforms

```bash
gtkwave dump_file.vcd
```

### Crucial Signal Hierarchy & Net Names

When inspecting waveforms in GTKWave, navigate down this path:
`TOP` -> `sim_top` -> `DUT` (`top_tile`) -> `subtile_inst` -> `sargantana_inst` (`top_drac`) -> `datapath_inst`

#### Key Taps for Tracking Dataflow:

1. **Clocks & Resets**:
   - `tb_clk`: Global clock.
   - `dut_rstn`: Active-low master reset.
   - `tile_rstn`: Internal core reset (incorporates soft resets and debug halts).

2. **Program Counter & Fetch**:
   - `datapath_inst.pc_if_1`: Active PC generated by Fetch 1.
   - `datapath_inst.req_cpu_icache_o.req_paddr`: Address sent to I-Cache.
   - `datapath_inst.resp_icache_cpu_i.valid`: I-Cache data valid signal.
   - `datapath_inst.resp_icache_cpu_i.data`: 512-bit cacheline returning to core.

3. **Instruction Decoding & Queue**:
   - `datapath_inst.pc_id`: PC of decoded instruction.
   - `datapath_inst.stage_if_2_id_q.inst`: 32-bit raw instruction word.
   - `datapath_inst.valid_id`: Instruction valid in decode.
   - `datapath_inst.control_int.stall_id`: Backpressure stalling decode.

4. **Rename & Dispatch (IR)**:
   - `datapath_inst.ir_stage_inst.rename_table_inst.map_table`: Architectural to physical register mapping table.
   - `datapath_inst.ir_stage_inst.free_list_inst.free_reg`: Available physical register pool.
   - `datapath_inst.graduation_list_inst.tail_ptr` / `head_ptr`: ROB allocation pointers.

5. **Register Read & Execution (RR -> EXE)**:
   - `datapath_inst.regfile_inst.mem`: Physical register file array.
   - `datapath_inst.stage_rr_exe_q.rs1_data` / `rs2_data`: Operands dispatched to execution units.
   - `datapath_inst.exe_stage_inst.alu_result`: Computed result from ALU.

6. **Memory Subsystem (LSU & HPDCache)**:
   - `datapath_inst.req_cpu_dcache_o.valid`: LSU memory request valid.
   - `datapath_inst.req_cpu_dcache_o.data_rs1`: Base address for load/store.
   - `datapath_inst.resp_dcache_cpu_i.valid`: D-Cache response valid.
   - `datapath_inst.resp_dcache_cpu_i.data`: Read data returning from D-Cache.

7. **Commit & Graduation**:
   - `datapath_inst.commit_valid[1:0]`: Up to 2 instructions committing simultaneously.
   - `datapath_inst.instruction_to_commit[0]`: Metadata for instruction retiring at slot 0.

## 7. The 4-Level Dataflow Debugging Methodology

When modifying RTL or diagnosing unexpected behavior, **never start by guessing in the waveforms**. Use this hierarchical recipe:

```
[Level 1: Terminal Summary] ---> Did test pass? Inspect PMU cycle counts.
           |
           v (If failed or stalled)
[Level 2: Commit Log]       ---> Find the exact PC and instruction where failure began.
           |
           v
[Level 3: Konata Trace]     ---> See which stage stalled (F1, D, Q, I, R, M, or ROB).
           |
           v
[Level 4: Waveform Probe]   ---> Open GTKWave at the exact cycle to inspect control wires.
```

### Concrete Walkthrough Example: Tracking a Modified Instruction

Suppose you add or modify an instruction in the [[decoder]]:

1. **Step 1: Run with full logs enabled**:
   ```bash
   ./sim +load=benchmarks/benchmarks/custom_tests.riscv \
         +commit_log=test_sig.txt \
         +konata_dump=test_konata.txt \
         +deadlock-cycles=500
   ```
2. **Step 2: Inspect `test_sig.txt`**:
   Search for your target instruction PC:

   ```bash
   grep -n "0x0000000080000120" test_sig.txt
   ```

   - If present: check whether the destination register and value match your expectation.
   - If absent: the core either trapped earlier or deadlocked before committing.

3. **Step 3: Open `test_konata.txt` in Konata**:
   Jump to that PC. Did the instruction get stuck in `Q` (Rename blocked)? In `R` (waiting on scoreboards)? In `M` (D-Cache not returning)?
4. **Step 4: Probe exact cycle in GTKWave**:
   If Konata shows a stall at cycle 1450, run:
   ```bash
   ./sim +load=benchmarks/benchmarks/custom_tests.riscv +vcd=focus.vcd +start-vcd-cycles=1400
   gtkwave focus.vcd
   ```
   Add `stage_iq_ir_q` and `control_int.stall_*` signals to pinpoint the root cause down to the exact gate.

## Related

[[pipeline_overview]] · [[instruction_dataflow]] · [[datapath]] · [[control_unit]] · [[graduation_list]] · [[free_list]] · [[decoder]] · [[drac_pkg]]
