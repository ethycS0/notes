
`branch_predictor` serves as the top-level branch prediction wrapper instantiated in [[if_stage_1|Fetch Stage 1]]. Because instructions are not yet decoded during fetch, this module acts as a fast tag filter to detect whether a fetched PC corresponds to a known branch instruction.

### **Branch Identification Table**

* Data Structures: 128-entry tag table `is_branch_table` and valid bit array `is_branch_table_valid`.
* Lookup Logic: Slices `pc_fetch_i` using index bits `[MOST_SIGNIFICATIVE_INDEX_BIT_BP:LEAST_SIGNIFICATIVE_INDEX_BIT_BP]` to read stored tags. Asserts `is_branch_prediction` high when tag matches `pc_fetch_i[39:MOST_SIGNIFICATIVE_INDEX_BIT_BP+1]` and the valid bit is set.
* Updates: When an instruction resolves in Execution (`is_branch_EX_i == 1`), the tag of `pc_execution_i` is written into `is_branch_table` at the indexed entry, setting valid to 1.

### **Submodule Integration**

* Instantiates [[bimodal_predictor|Bimodal Predictor]] to handle direction (taken/not-taken) and target address prediction.
* Interfaces with [[return_address_stack|Return Address Stack]] (RAS) for subroutine return speculation.
* Output Assignments: Drives `branch_predict_is_branch_o` from the tag match signal, while forwarding `bimodal_predict_taken` to `branch_predict_taken_o` and `bimodal_predict_addr` to `branch_predict_addr_o`.