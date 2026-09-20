
`if_stage_1` handles Program Counter (PC) generation, branch prediction lookups, front-end exception pre-checks, and issues instruction fetch requests to the instruction cache (I-cache). It operates in stage F1 of the pipeline and communicates with [[pipeline|Sargantana Pipeline]].

### **PC Generation Logic**

The module computes `next_pc` using a combinational priority multiplexer driven by `cu_if_i.next_pc`:

* `NEXT_PC_SEL_KEEP_PC`: Retains `pc` (used during pipeline stalls).
* `NEXT_PC_SEL_BP_OR_PC_4`: Evaluates branch prediction output from [[branch_predictor|Branch Predictor]]. If `branch_predict_is_branch && branch_predict_taken` is active, `next_pc` takes `branch_predict_addr` (BTB target). Otherwise, it evaluates to `pc + 4` using `trunc_pc_sum`.
* `NEXT_PC_SEL_JUMP` / `NEXT_PC_SEL_DEBUG`: Overrides current fetch with redirect address `pc_jump_i` coming from Execution stage mispredictions, jumps, traps, or debug triggers.

On `posedge clk_i`, `pc` latches `next_pc`. Reset sets `pc` to `reset_addr_i`.

### **Early Exception Detection**

Before accessing the I-cache, three parallel combinational checks run:

* Misalignment: Checks `|pc[1:0]`. Asserts `ex_addr_misaligned_int` if non-zero.
* Address Fault: Checks sign-extension bits `pc[63:VIRT_ADDR_SIZE-1]` when translation is enabled (`en_translation_i`), or validates mapped physical regions via `is_inside_mapped_sections` when translation is off.
* Guest Page Fault: Checks `|pc[63:PHY_VIRT_MAX_ADDR_SIZE-1]` under hypervisor G-stage translation rules.

### **I-Cache Request & Pipeline Output**

* `req_cpu_icache_o.valid`: Driven high if no internal exceptions (`ex_addr_misaligned_int`, `ex_if_addr_fault_int`, `ex_if_guest_page_fault_int`) and no stalls (`stall_i`, `stall_debug_i`) are active. Sends `pc[PHY_VIRT_MAX_ADDR_SIZE-1:0]` as `vaddr`.
* `fetch_o`: Bundles `pc`, branch metadata (`is_branch`, `decision`, `pred_addr`), and exception info (`cause`, `valid`, `origin`, `origin2`, `gva`) to pass downstream to [[if_stage_2|Fetch Stage 2]].