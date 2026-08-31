
> **Primary Resource:** `meltdown.pdf` — *Meltdown: Reading Kernel Memory from User Space* (Lipp et al., USENIX Security 2018)

---
### 1. Introduction & Authors
- [[introduction]]

### 2. Out-of-Order Execution 
- [[general-concept]] — Core concepts, in-order vs out-of-order execution, architectural vs microarchitectural state.
- [[tomasulo-algorithm]] — Theoretical breakdown of dynamic scheduling, reservation stations, register renaming, CDB, and RAW/WAR/WAW hazard elimination.
- [[intel-x86-pipeline]] — Intel x86 microarchitecture, Front-End $\mu$OP decoding, Execution Engine, ROB, reservation stations, and memory subsystem.
- [[exception-handling-and-retirement]] — Reorder Buffer (ROB) retirement, transient execution window, exception deferral, microarchitectural cache state leakage.

### 3. Memory Addressing & Caches (`addressing-and-caches/`)
- [[virtual-and-physical-addressing]] — Virtual memory, physical memory, page translation tables (`U/S` flag bit), kernel space mapping, and Linux `physmap`.
- [[cpu-caches]] — Memory hierarchy (L1/L2/LLC), 64-byte cache lines, hit/miss timing differentials, and microarchitectural state persistence.
- [[cache-side-channel-attacks]] — Cache covert channels, Flush+Reload step-by-step mechanism (`clflush`, transient load, `rdtscp` timing), 4096-byte stride page alignment, and Evict+Reload / Prime+Probe alternatives. Includes generated sequence diagram (`flush-reload-attack.jpg`).
