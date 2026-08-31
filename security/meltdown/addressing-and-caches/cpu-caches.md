[[INDEX]] | [[addressing-and-caches/virtual-and-physical-addressing]] | [[addressing-and-caches/cache-side-channel-attacks]]

CPU caches are small, ultra-fast hardware memories integrated directly into the processor core to bridge the massive speed gap between fast CPU execution units and slow main system RAM. Caches operate transparently to software using a hierarchy of levels: Level 1 (L1 Data and Instruction caches, fastest, ~4 clock cycles latency, per-core), Level 2 (L2 cache, ~12 cycles latency, usually per-core), and Last Level Cache (LLC / L3, ~40-60 cycles latency, shared across CPU cores), compared to main DRAM access which takes over 200 clock cycles.

Data in memory is moved into caches in fixed-size blocks called cache lines (typically 64 bytes on x86 architectures). When the CPU executes a memory read instruction, it queries the L1 data cache using the address tag. If the target cache line is present (a Cache Hit), the data is returned immediately to the register file. If the cache line is absent (a Cache Miss), the CPU issues a bus request to fetch the entire 64-byte line from lower cache levels or main DRAM, inserting it into the L1 cache for subsequent access.

The core microarchitectural property exploited by side-channel attacks is that cache state is stateful and persistent across instruction rollbacks. While CPU architectural registers are restored to pristine values when a transient instruction is discarded during an exception or misprediction, the cache line loaded during that transient execution window remains resident in the L1 data cache. Because a cache hit is orders of magnitude faster to read than a cache miss, an attacker can measure access time to deduce whether a specific memory address was loaded into the cache during transient execution.

---
*Next Note: [[addressing-and-caches/cache-side-channel-attacks]]*
