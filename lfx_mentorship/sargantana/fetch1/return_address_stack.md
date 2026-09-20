
`return_address_stack` (RAS) is a 16-entry hardware shadow stack used to predict subroutine return addresses (`ret` / `JALR`) speculatively, bypassing memory and register file latencies. It connects into the front-end branch prediction framework controlled by [[branch_predictor|Branch Predictor]].

### **Data Structures & Pointers**

* Memory: 16-entry array `address_stack` storing 64-bit target PCs.
* Pointers: `head_pointer` (4-bit counter pointing to next open push slot) and `output_pointer` (combines as `head_pointer - 1`, pointing to the current top of stack).

### **Stack Operations**

* Push (`push_i && !pop_i`): Writes `pc_execution_i` to `address_stack[head_pointer]` and increments `head_pointer` using 4-bit modular addition `trunc_ras_sum(head_pointer + 1)`.
* Pop (`pop_i && !push_i`): Decrements `head_pointer` using `trunc_ras_sum(head_pointer - 1)`.
* Simultaneous Push & Pop (`push_i && pop_i`): Writes `pc_execution_i` directly to `address_stack[head_pointer]` without changing `head_pointer` (handles tail-call scenarios).
* Output: `return_address_o` continuously outputs `address_stack[output_pointer]`.