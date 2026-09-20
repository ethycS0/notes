
Sargantana is a 7-stage decoupled, in-order issue, out-of-order write-back RISC-V core. The architecture divides pipeline responsibilities across a decoupled front-end and back-end structure separated by an instruction queue.

![[sargantana_pipeline_diagram.png]]

---

## **Front-End Overview**

The front-end operates speculatively to continuously supply instructions to the rest of the processor. Its goal is to predict branch directions, look up target addresses, and fetch instruction words from memory without causing pipeline stalls.

### **[[if_stage_1|Fetch Stage 1]]**

Calculates next_pc using sequential +4, branch predictions, or back-end redirect jumps (pc_jump_i). It queries [[branch_predictor|branch_predictor]], which uses a 128-entry tag table (is_branch_table) to identify known branches without instruction decoding. Direction and target predictions are handled by [[bimodal_predictor|bimodal_predictor]] using a 128-entry Pattern History Table (PHT) of 2-bit counters and a Branch Target Buffer (BTB) for zero-cycle speculative lookups. Subroutine return addresses are predicted via the 16-entry [[return_address_stack|return_address_stack]]. Parallel checks for PC misalignment (pc[1:0] != 0) and physical section mapping violations are performed before sending virtual address requests (vaddr) to the instruction cache.

### **[[if_stage_2|Fetch Stage 2]]**

Receives incoming 32-bit instruction payloads from the cache and aligns them with PC and branch metadata forwarded from IF1. It asserts backpressure stalls (stall_o) on cache misses or TLB delays to freeze the upstream PC. During downstream pipeline stalls (stall_i), it holds instruction payloads in a 1-entry skid buffer register (resp_icache_cpu_q) to prevent data loss. It also merges exceptions by prioritizing early IF1 address faults over MMU instruction page faults (INSTR_PAGE_FAULT) and guest page faults (INSTR_GUEST_PAGE_FAULT) before dispatching data to the instruction queue.

---

## Back-End Overview
