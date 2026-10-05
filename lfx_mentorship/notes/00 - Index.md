## Map of Content (MOC)

1. **[[01 - Tooling & Simulation Workflow]]**
   - Comprehensive guide to Verilator and simulation binaries (`./sim`).
   - Complete reference for simulation plusargs (`+load`, `+commit_log`, `+konata_dump`, `+vcd`, etc.).
   - Dissecting and using the **Commit Log** (`signature.txt`).
   - Dissecting and visualizing pipeline traces in **Konata** (`konata.txt`).
   - Waveform dumping and critical signal tapping in GTKWave / Surfer.
   - **4-Level Dataflow Debugging Methodology** (with concrete commands & recipes).

2. **[[02 - Core Tile Architecture]]**
   - High-level composition: `top_tile`, `sargantana_subtile`, and `top_drac`.
   - The memory subsystem: L1 Instruction Cache, Uncacheable Buffer (`nc_icache_buffer`), HPDCache (L1 Data Cache with MSHRs & coalescing write buffers), and L2/Memory model.
   - Memory Management Unit (MMU): SV39/SV48 virtual memory, iTLB, dTLB, and the Hardware Page Table Walker (PTW).
   - Uncore & Peripherals: BootROM, RISC-V Debug Module (JTAG / DTM / SRI), and PMU performance counters.

3. **[[03 - Sargantana Pipeline & Datapath]]**
   - Core microarchitecture paradigm: In-Order Fetch/Decode, Register Renaming (PRF-based), Out-of-Order Multi-Cycle Execution, In-Order 2-wide Graduation (ROB).
   - Stage-by-stage functional breakdown:
     - **IF1 & IF2**: Next-PC selection, bimodal branch prediction, RAS, and instruction slicing.
     - **ID**: Instruction decoding, immediate extraction, instruction queue, and vector configuration (`vset`).
     - **IR**: Free list management, rename table (RAT), graduation list allocation, and speculative checkpointing.
     - **RR**: Physical register file reads, scoreboarding, and forwarding networks.
     - **EXE**: Execution units (ALU, Branch Unit, Mul, Div, Load-Store Unit / LSQ, FPU, Vector VAGU/SIMD, CSR).
     - **WB & Commit**: Physical register writeback, Graduation List commit, architectural retirement, exception handling.
   - Central Control Unit: Stalls, flushes, and hazard mitigation.

4. **[[04 - Instruction Dataflow Walkthroughs]]**
   - Concrete, step-by-step lifecycle of instructions across all stages:
     - **Trace 1**: Simple ALU Operation (`addi s0, zero, 1`).
     - **Trace 2**: Memory Load/Store (`ld` & `sd`) through LSQ and HPDCache.
     - **Trace 3**: Branch Prediction, Misprediction detection, and Checkpoint Rollback.
