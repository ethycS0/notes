
The **Sargantana `core_tile`** encapsulates a complete high-performance 64-bit RISC-V computing tile. It integrates the **Sargantana** out-of-order execution core with dedicated L1 instruction and data caches, a comprehensive Memory Management Unit (MMU), uncacheable memory buffers, and standard system interconnect interfaces.

## 1. Top-Level Hierarchy & Decomposition

The tile hierarchy is cleanly partitioned into three nested abstraction layers:

```
+--------------------------------------------------------------------------------------+
| top_tile.sv                                                                          |
|                                                                                      |
|  +--------------------+  +----------------------+  +-------------------------------+ |
|  | sargantana_top_    |  | nc_icache_buffer     |  | hpdcache                      | |
|  | icache (L1 I$)     |  | (BootROM & UC Fetch) |  | (OpenHardware L1 D$)          | |
|  +---------^----------+  +----------^-----------+  +---------------^---------------+ |
|            |                        |                              |                 |
|  +---------v------------------------v------------------------------v---------------+ |
|  | sargantana_subtile.sv                                                           | |
|  |                                                                                 | |
|  |   +-----------------------+    +--------------------+    +--------------------+ | |
|  |   | icache_interface      |    | dcache_interface   |    | bsc_mmu_hpdc_      | | |
|  |   | (Fetch <-> I-Cache)   |    | (LSU <-> HPDCache) |    | adapter (PTW <->   | | |
|  |   |                       |    | (SID 1)            |    | HPDCache SID 0)    | | |
|  |   +-----------^-----------+    +---------^----------+    +---------^----------+ | |
|  |               |                          |                         |            | |
|  |   +-----------v--------------------------v-------------------------v----------+ | |
|  |   | top_drac.sv (Processor Core)                                              | | |
|  |   |                                                                           | | |
|  |   |   +-------------------+  +--------------------+  +--------------------+   | | |
|  |   |   | datapath.sv       |  | csr_bsc.sv         |  | bsc_mmu.sv         |   | | |
|  |   |   | (7-stage pipeline)|  | (Privileged CSRs)  |  | (iTLB, dTLB, PTW)  |   | | |
|  |   |   +-------------------+  +--------------------+  +--------------------+   | | |
|  |   +---------------------------------------------------------------------------+ | |
|  +---------------------------------------------------------------------------------+ |
+--------------------------------------------------------------------------------------+
```

### Module Responsibilities
- **`top_tile.sv`**: The outermost physical block boundary. Instantiates the L1 caches, uncacheable buffers, and maps external system buses (L2 miss/grant ports, memory-mapped I/O, interrupts, and JTAG).
- **`sargantana_subtile.sv`**: Encapsulates the core together with its protocol adapter shims (`icache_interface`, `dcache_interface`, and `bsc_mmu_hpdc_adapter`).
- **`top_drac.sv`**: The core-level integration point. Binds the datapath execution pipeline to the CSR control block, MMU translation logic, and hardware performance counters.

## 2. Memory Subsystem

The tile features separate, highly specialized L1 instruction and data memory paths designed to prevent cache pollution and latency bottlenecks.

```
                      +-------------------+
                      |   datapath.sv     |
                      +---+-----------+---+
                          |           |
            (Fetch Req)   |           | (Load/Store Req)
                          v           v
                 +------------+   +------------+
                 |   iTLB     |   |   dTLB     |
                 +-----+------+   +-----+------+
                       |                |
         [VIPT Lookup] |                | [Direct Physical]
                       v                v
            +---------------+     +------------+  (Port 0 / SID 0)  +------------+
            |  L1 I-Cache   |     |  HPDCache  |<-------------------| MMU PTW    |
            +-------+-------+     +-----+------+                    +------------+
                    | (Acquire/Grant)   | (Read Miss / Writeback)
                    v                   v
            +----------------------------------+
            |      L2 Interconnect / NoC       |
            +----------------------------------+
```

### A. L1 Instruction Cache (`sargantana_top_icache`)

- **Structure**: 4-way set-associative cache with 64 sets.
- **Cacheline Size**: 512 bits (64 bytes) per line, matching a wide fetch burst. Total capacity = 16 KB.
- **Translation Scheme**: Virtually Indexed, Physically Tagged (**VIPT**). The cache set index is looked up in parallel with the iTLB physical translation, minimizing fetch latency.
- **Hit/Miss Control**: Misses trigger an acquire request (`io_mem_acquire_valid`) to L2, which refills via beat grants (`io_mem_grant_bits_data`).

### B. Non-Cacheable I-Cache Buffer (`nc_icache_buffer`)

Standard caches must not cache memory-mapped device registers, BootROM sequences, or debug program buffers.
- Sits between the core fetch stage and the I-Cache.
- Evaluates the fetch address against known I/O and BootROM memory ranges (`InitBROMBase` to `InitBROMEnd`, `DebugProgramBufferBase`).
- **Action**: When an uncacheable access is detected, it bypasses the L1 I-Cache entirely, routes the request directly to the BootROM/SRI bus, and returns the aligned instruction stream back to Fetch Stage 2.

### C. High-Performance L1 Data Cache (`hpdcache`)

Developed by the OpenHardware Group, CEA, and BSC, the **HPDCache** is a high-bandwidth, non-blocking L1 data cache.
- **Non-blocking Execution with MSHRs**: Contains multiple Miss Status Holding Registers (MSHRs). When a load or store misses, the cache does not freeze; it continues serving hits-under-miss for other operations.
- **Coalescing Write Buffer (`wbuf`)**: Aggregates adjacent byte/word store requests into single line writes before pushing them out to L2, optimizing memory bus utilization.
- **Dual Requesters (Arbiter)**:
  - **Requester 0 (SID 0)**: Hardware Page Table Walker (`bsc_mmu_hpdc_adapter`). Allows translation table walks to read page table entries directly through the cache hierarchy.
  - **Requester 1 (SID 1)**: Core Load-Store Unit (`dcache_interface`). Serves ordinary scalar, floating-point, and vector loads/stores.

## 3. Memory Management Unit (`bsc_mmu`)

The core supports standard RISC-V **SV39** (and configurable SV48) page-based virtual memory architectures with hypervisor and supervisor mode protections.

### Key MMU Components:

1. **Instruction TLB (iTLB)**:
   - Dedicated translation buffer for the instruction fetch pipeline.
   - Low-latency lookups; on a hit, delivers the Physical Page Number (PPN) directly to the I-Cache tag checker.
2. **Data TLB (dTLB)**:
   - Dedicated translation buffer for memory loads, stores, and atomics.
   - Checked during memory address generation in the Execute stage.
3. **Hardware Page Table Walker (PTW)**:
   - When either the iTLB or dTLB experiences a miss, the PTW autonomously takes over.
   - Reads the root page table pointer from `satp` (or `vsatp`/`hgatp` under virtualization).
   - Generates read transactions over HPDCache Requester Port 0 to traverse the radix tree levels.
   - Upon locating the leaf PTE (Page Table Entry), it inserts the translation into the offending TLB and resumes the stalled core pipeline.
   - Flags Access Faults or Page Faults directly to the CSR exception handler if permissions fail.

## 4. Control, Status & Uncore Blocks

### A. Privileged Control & Status Registers (`csr_bsc.sv`)

- Implements RISC-V Privileged Specification v1.11 / v1.12.
- Handles User (`U`), Supervisor (`S`), and Machine (`M`) privilege levels, plus virtualization mode (`V`).
- Implements trap redirection vectors (`mtvec`, `stvec`, `vstvec`), exception cause reporting (`mcause`, `scause`), and fault addresses (`mtval`, `stval`).
- Synchronizes with the core pipeline during serializing instructions (`CSRRW`, `CSRRS`, `CSRRC`, `sret`, `mret`, `sfence.vma`).

### B. BootROM Subsystem

- Mapped at address `0x0000000000000100`.
- Holds the initial boot code (`bootrom.hex`).
- In the simulation testbench, the BootROM sets up basic stack/pointer registers and immediately jumps to the entry point `0x80000000` (where user code or benchmarks are loaded).
- Default Trap Vector is located at offset `+0x40` (`0x0000000000000140`). Unhandled exceptions in bare-metal tests trap here to a `wfi` loop.

### C. Debug Module & JTAG Interface

- Complies with the **RISC-V Debug Specification 0.13**.
- Driven in simulation via `SimJTAG` and the Debug Transport Module (`dtm`).
- Connects through asynchronous Gray-code CDC FIFOs to the core Debug Module (`riscv_dm`).
- Provides hardware run-control (halt, resume, single-step) and provides access to the register file and Program Buffer via the Synchronous Register Interface (**SRI**).

### D. Performance Monitoring Unit (PMU) & HPM Counters

- Located in `top_drac.sv` (`hpm_counters.sv`).
- Includes **29 64-bit performance counters** (`mhpmcounter3` - `mhpmcounter31`).
- Capable of counting up to **40 distinct microarchitectural events**, including:
  - Branch mispredictions, taken branches.
  - Stage stall cycles (`stall_if`, `stall_id`, `stall_rr`, `stall_exe`, `stall_wb`).
  - Cache hits, misses, fills, and kills.
  - Structural stalls (Graduation List Full, Free List Empty).
  - Data dependency interlocks and store-forwarding delays.
  - iTLB, dTLB, and PTW access/miss rates.
