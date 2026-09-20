
`bimodal_predictor` implements direction prediction and branch target address storage. It is instantiated within [[branch_predictor|Branch Predictor]] and operates combinationally during [[if_stage_1|Fetch Stage 1]] to avoid fetch bubbles.

### **Data Structures**

* Pattern History Table (PHT): Array `pattern_history_table` containing 128 entries of 2-bit saturating counters (`2'b00` Strongly Not-Taken, `2'b01` Weakly Not-Taken, `2'b10` Weakly Taken, `2'b11` Strongly Taken).
* Branch Target Buffer (BTB): Array `branch_target_buffer` containing 128 entries of `PHY_VIRT_MAX_ADDR_SIZE + 1` bits storing past branch target addresses.

### **Fetch Lookup & Prediction**

* PHT Read: Read state `readed_state_pht` indexed by `pc_fetch_i`. Direction prediction output `bimodal_predict_taken_o` is driven by the MSB `readed_state_pht[1]` (`1` = Taken, `0` = Not-Taken).
* BTB Read: Target address `short_pred_addr` read from `branch_target_buffer` using `pc_fetch_i`. Outputs sign-extended address `bimodal_predict_addr_o` to [[if_stage_1|Fetch Stage 1]].

### **Predictor Training State Machine**

When `is_branch_EX_i` is asserted by Execution:

* `past_state_pht` is read from PHT at index `pc_execution_i`.
* `new_state_to_pht` evaluates using 2-bit saturating counter rules: increments on actual branch taken (`branch_taken_result_exec_i == 1`) up to `2'b11`, decrements on not-taken down to `2'b00`.
* PHT updates with `new_state_to_pht` and BTB updates with `branch_addr_result_exec_i` on `posedge clk_i`.