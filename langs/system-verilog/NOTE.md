## Fundamental Constructs

### Module

The `module` is the basic building block in SystemVerilog. Unlike legacy HDLs, port lists can use ANSI-style declarations combining direction, data type, and net structure in place.

```systemverilog
module alu #(
    parameter int XLEN = 32
) (
    input  logic [XLEN-1:0] a,
    input  logic [XLEN-1:0] b,
    input  logic [3:0]      op,
    output logic [XLEN-1:0] res,
    output logic            zero
);
    always_comb begin
        case (op)
            4'b0000: res = a + b;
            4'b0001: res = a - b;
            4'b0010: res = a & b;
            4'b0011: res = a | b;
            default: res = '0;
        endcase
        zero = (res == '0);
    end
endmodule

```

### Interface

An `interface` encapsulates signals, directionality (`modport`), and timing checks (`clocking block`) into a single composite channel to connect design blocks clean of pin-swapping bugs.

```systemverilog
interface riscv_bus_if #(parameter int XLEN = 32) (input logic clk, rst_n);
    logic [XLEN-1:0] addr;
    logic [XLEN-1:0] wdata;
    logic [XLEN-1:0] rdata;
    logic            mem_read;
    logic            mem_write;
    logic            ready;

    // Master View (CPU Core)
    modport master (
        output addr, wdata, mem_read, mem_write,
        input  rdata, ready, clk, rst_n
    );

    // Slave View (Data Memory)
    modport slave (
        input  addr, wdata, mem_read, mem_write, clk, rst_n,
        output rdata, ready
    );
endinterface

```

### Procedural Blocks

SystemVerilog explicitly enforces design intent via specialized procedural blocks:

* **`always_comb`**: Replaces general combinational processes. Automatically evaluates at $t=0$, infers full sensitivity, and throws compile errors if latches are inferred or variables have multiple drivers.
* **`always_ff @(posedge clk or negedge rst_n)`**: Dedicated block for sequential flip-flops. Restricts assignments exclusively to non-blocking (`<=`).
* **`always_latch`**: Dedicated block for intentional level-sensitive latches.
* **`initial`**: Executes once at simulation startup; used strictly in testbenches or ROM initializations.

---

## Identifiers

### Generic Syntax

```systemverilog
// Variable and Object Declaration Standard
data_type [vector_bounds] identifier_name = initial_value;

```

**Rules for Identifiers:**

* Standard characters: letters (`a-z`, `A-Z`), digits (`0-9`), and underscores (`_`).
* First character must be a letter or an underscore.
* Case-sensitive (`DataBus` and `databus` are distinct).
* Keywords cannot be used unless escaped with a backslash (e.g., `\module `).

**Types:**

1. **Standard Identifiers**: `clk`, `alu_result`, `instr_mem`.
2. **Escaped Identifiers**: Begin with `\` and end with a whitespace character (e.g., `\bus+1 `).
3. **System Tasks/Functions**: Prefixed with `$` (e.g., `$display`, `$clog2`, `$finish`).

---

## Data Objects & Data Types

### Data Objects

* **`parameter`**: Compile-time constant per module instance.
* **`localparam`**: Local constant within a module/package; cannot be overridden directly from instantiation.
* **`const`**: Run-time read-only variable initialized during execution.
* **`var`**: Explicit keyword defining a variable object (optional when combined with types like `logic`).
* **`wire`**: Net type representing physical connectivity; supports multiple drivers (resolves collisions like high-Z/conflict).

### Data Types: 4-State vs. 2-State

SystemVerilog classifies data types into 4-state logic (`0`, `1`, `X`, `Z`) for physical modeling and 2-state logic (`0`, `1`) for high-performance verification/abstract data models.

```systemverilog
// 4-State Types (Default to 'X')
logic [31:0] data_bus; // Preferred single-driver hardware type
integer      index;    // 32-bit signed integer
time         t_stamp;  // 64-bit unsigned time quantity

// 2-State Types (Default to '0')
bit   [7:0]  byte_val; // Unsigned 8-bit bit-vector
byte         s_byte;   // 8-bit signed integer
int          counter;  // 32-bit signed integer
longint      big_int;  // 64-bit signed integer
shortint     s_int;    // 16-bit signed integer

```

### Composite Types

**1. Packed Arrays**
Stored as a contiguous sequence of bits in memory. Can be sliced, used in arithmetic, and assigned in a single step.

```systemverilog
logic [3:0][7:0] packed_dword; // 32 bits total, grouped as 4 bytes

```

**2. Unpacked Arrays**
Represent multi-element storage spaces (e.g., Register Files, RAMs).

```systemverilog
logic [31:0] reg_file [0:31]; // 32 individual 32-bit registers

```

**3. Packed Structures**
Bit-aligned fields stored without padding; synthesizes directly to hardware vectors.

```systemverilog
typedef struct packed {
    logic [6:0]  funct7;
    logic [4:0]  rs2;
    logic [4:0]  rs1;
    logic [2:0]  funct3;
    logic [4:0]  rd;
    logic [6:0]  opcode;
} riscv_r_type_t;

```

**4. Enumerations**
Provides type safety for FSM encoding and status states.

```systemverilog
typedef enum logic [1:0] {
    IDLE  = 2'b00,
    FETCH = 2'b01,
    EXEC  = 2 me10,
    WB    = 2'b11
} state_e;

```

---

## Operators

### Operator Precedence

| Precedence | Operators | Description |
| --- | --- | --- |
| **Highest** | `()`, `[]`, `::`, `.` | Grouping, Indexing, Scope, Member Access |
|  | `+`, `-`, `!`, `~`, `&`, `~&`, `|`, `~|`, `^`, `~^` | Unary Operators, Reduction Operators |
|  | `**` | Exponentiation |
|  | `*`, `/`, `%` | Binary Multiplication, Division, Modulus |
|  | `+`, `-` | Binary Addition, Subtraction |
|  | `<<`, `>>`, `<<<`, `>>>` | Logical & Arithmetic Shifts |
|  | `<`, `<=`, `>`, `>=` | Relational Comparisons |
|  | `==`, `!=`, `===`, `!==`, `==?`, `!=?` | Equality, Case Equality, Wildcard Equality |
|  | `&` | Bitwise AND |
|  | `^`, `~^` | Bitwise XOR / XNOR |
|  | `|` | Bitwise OR |
|  | `&&` | Logical AND |
|  | `||` | Logical OR |
|  | `?:` | Ternary Conditional Operator |
| **Lowest** | `{}` , `{n{}}` | Concatenation and Replication |

### Key Equality Operator Differences

* **Logical Equality (`==`, `!=`)**: Returns `1'bX` if either operand contains `X` or `Z` values.
* **Case Equality (`===`, `!==`)**: Compares literal values bit-for-bit including `X` and `Z`. Returns strict `1'b1` or `1'b0` (Non-synthesizable, used in simulation).
* **Wildcard Equality (`==?`, `!=?`)**: Treats `X` and `Z` bits in the right-hand operand as "don't cares".

```systemverilog
// RISC-V Field Extraction & Immediate Reconstruction Example
logic [31:0] instr;
logic [31:0] imm_i;

assign imm_i = {{20{instr[31]}}, instr[31:20]}; // Sign-extension via replication & concatenation

```

---

## Modeling

Modeling defines the abstraction level used to represent digital logic.

| Style | Description | Key Constructs | Typical Use-case |
| --- | --- | --- | --- |
| **Behavioral** | High-level algorithmic logic, control-flow | `always_comb`, `always_ff`, `case` | FSMs, CPUs, Control Units |
| **Data-Flow** | Direct signal dependencies via logic equations | Continuous assignments (`assign`) | ALUs, Decoders, Multiplexers |
| **Structural** | Hierarchy of instantiated interconnects | Module instantiations, `interface` | Chip top-level, Core integration |

### Behavioral Modeling Example

```systemverilog
always_comb begin
    case (alu_ctrl)
        4'b0000: alu_out = operand_a + operand_b;
        4'b0001: alu_out = operand_a - operand_b;
        default: alu_out = '0;
    endcase
end

```

### Data-Flow Modeling Example

```systemverilog
assign mem_write_enable = (opcode == 7'b0100011) && (funct3 == 3'b010);

```

### Structural Modeling Example

```systemverilog
module riscv_core (
    input logic clk, rst_n,
    riscv_bus_if.master bus
);
    logic [31:0] pc, instr, alu_res;

    pc_reg   u_pc (.clk(clk), .rst_n(rst_n), .next_pc(pc + 4), .current_pc(pc));
    alu_unit u_alu (.a(pc), .b(32'd4), .op(4'b0000), .res(alu_res));
endmodule

```

---

## Concurrent Statements

Concurrent statements model hardware elements that operate continuously and simultaneously in physical silicon.

### 1. Continuous Assignments (`assign`)

Drives net/logic structures continuously whenever right-hand side signals evaluate to a new value.

```systemverilog
assign zero_flag = (alu_result == 32'b0);

```

### 2. Procedural Blocks (`always_comb`, `always_ff`)

Executes concurrently relative to other blocks in the module hierarchy.

```systemverilog
always_ff @(posedge clk or negedge rst_n) begin
    if (!rst_n) pc <= '0;
    else        pc <= next_pc;
end

```

### 3. Generate Blocks

Constructs repetitive or conditional design blocks based on compile-time parameters.

```systemverilog
generate
    genvar i;
    for (i = 0; i < 32; i++) begin : gen_reg_file
        always_ff @(posedge clk or negedge rst_n) begin
            if (!rst_n) reg_matrix[i] <= '0;
            else if (we && (write_addr == i) && (i != 0))
                reg_matrix[i] <= write_data;
        end
    end
endgenerate

```

### 4. Concurrent Assertions (SVA)

Continuously monitors signals across clock edges to catch protocol violations dynamically during runtime.

```systemverilog
assert property (@(posedge clk) mem_read |-> ##[1:2] mem_ready);

```

---

## Sequential Statements

Sequential statements execute step-by-step inside procedural blocks (`always`, `initial`, `tasks`, `functions`).

### Blocking vs. Non-Blocking Assignments

* **Blocking (`=`):** Evaluates and updates the target variable immediately before proceeding to the next statement. Use strictly inside `always_comb` and functions.
* **Non-Blocking (`<=`):** Schedules updates to occur at the end of the current simulation time step (NBA region). Use strictly inside `always_ff` blocks to prevent race conditions.

### IF-ELSE & Unique/Priority Keywords

```systemverilog
// 'unique' enforces mutually exclusive conditions; warns if multiple cases hit or none hit without default.
unique if (state == IDLE) begin
    next_state = FETCH;
end else if (state == FETCH) begin
    next_state = EXEC;
end

// 'priority' enforces order evaluation; warns if no condition evaluates true.
priority if (irq[0]) handle_irq0();
else if (irq[1])    handle_irq1();

```

### CASE Statements (`unique`, `priority`, `casez`)

```systemverilog
always_comb begin
    unique case (opcode)
        7'b0110011: control_type = R_TYPE;
        7'b0010011: control_type = I_TYPE;
        7'b0100011: control_type = S_TYPE;
        default:    control_type = UNKNOWN;
    endcase
end

// casez treats '?' as a don't-care bit (useful for priority decoding)
always_comb begin
    casez (req_bus)
        4'b1???: grant = 2'b11;
        4'b01??: grant = 2'b10;
        4'b001?: grant = 2'b01;
        default: grant = 2'b00;
    endcase
end

```

### Loops

```systemverilog
// 1. Static bounds For Loop (Synthesizable)
for (int i = 0; i < 32; i++) begin
    bit_reversed[i] = data_in[31 - i];
end

// 2. Foreach Loop (Iterates native array dimensions)
foreach (reg_file[k]) begin
    reg_file[k] = '0;
end

// 3. Repeat & While Loops (Mainly for testbenches)
repeat (10) @(posedge clk);
while (!ready) @(posedge clk);

```

---

## Packages

Packages group globally accessible data types, constants, structures, and subprograms together, replacing legacy macro header includes.

```systemverilog
package riscv_pkg;
    localparam int XLEN = 32;

    typedef enum logic [6:0] {
        OP_LUI   = 7'b0110111,
        OP_AUIPC = 7'b0010111,
        OP_ADD   = 7'b0110011
    } opcode_e;

    function automatic logic [31:0] sign_extend_i(logic [11:0] imm);
        return {{20{imm[11]}}, imm};
    endfunction
endpackage : riscv_pkg

// Using the Package in RTL
module decoder import riscv_pkg::*; (
    input  logic [31:0] instr,
    output opcode_e     op_type
);
    assign op_type = opcode_e'(instr[6:0]);
endmodule

```

---

## Libraries & Simulation Mechanics

### 4-State Logic Interpretation

SystemVerilog built-in net and logic data types support 4 distinct state values:

* **`0`**: Logic zero / False.
* **`1`**: Logic one / True.
* **`X`**: Unknown / Uninitialized / Contention conflict.
* **`Z`**: High Impedance / Unconnected float.

### Stratified Event Queue

To prevent race conditions between design and testbench elements, the SystemVerilog simulator orders execution within a single time step:

1. **Active Region:** Evaluates blocking assignments (`=`), RHS of non-blocking assignments, and continuous assignments.
2. **Inactive Region:** Evaluates `#0` delayed statements (discouraged practice).
3. **NBA Region:** Updates LHS values of non-blocking assignments (`<=`).
4. **Observed Region:** Evaluates Concurrent Assertions (`assert property`).
5. **Reactive Region:** Executes testbench program blocks, stimulus generation, and verification pass/fail drivers.
6. **Postponed Region:** Reads final output values after all settling clears (`$strobe`, `$monitor`).

---

## Configurations

SystemVerilog configurations map specific modules, implementations, or gate-level representations to specific instances within a hierarchical design without altering the source RTL code.

```systemverilog
config cpu_config;
    design work.riscv_top;
    
    // Bind ALU instance to functional behavioral model
    default liblist work;
    instance riscv_top.core.alu_inst use work.alu_behavioral;
endconfig

```

---

## Subprograms: Functions & Tasks

Subprograms break procedural code into reusable routines.

* **Functions:** Execute in $0$ simulation time, return a value, and **cannot** contain time-consuming constructs (`@`, `#`, `wait`).
* **Tasks:** Can consume simulation time, return no direct single value (use `output` or `inout` arguments), and drive timing sequences.

```systemverilog
// Automatic function (Allocates memory dynamically per call; required for recursion)
function automatic logic [31:0] alu_add (input logic [31:0] a, b);
    return a + b;
endfunction

// Task Driving Memory Access
task automatic mem_write (
    input  logic [31:0] addr,
    input  logic [31:0] data,
    ref    riscv_bus_if bus
);
    @(posedge bus.clk);
    bus.addr      <= addr;
    bus.wdata     <= data;
    bus.mem_write <= 1'b1;
    wait (bus.ready);
    @(posedge bus.clk);
    bus.mem_write <= 1'b0;
endtask

```

### Function vs. Task Summary

| Feature | Function | Task |
| --- | --- | --- |
| **Simulation Time** | Zero time guaranteed | Can consume time (`#`, `@`, `wait`) |
| **Return Value** | Returns a single value | No return value (uses output args) |
| **Calls** | Can call other functions | Can call functions and other tasks |
| **Synthesis** | Synthesizable | Generally non-synthesizable (Testbenches) |

---

## Test Benches & Verification (SV-HVL)

SystemVerilog extends traditional design capabilities with Object-Oriented Verification concepts.

### 1. Classes & Randomization

```systemverilog
class riscv_transaction;
    rand bit [31:0] instr;
    rand bit [4:0]  rs1, rs2, rd;
    rand bit [6:0]  opcode;

    // Constrained Randomization
    constraint valid_rd { rd != 5'd0; } // x0 is read-only
    constraint r_type  { opcode == 7'b0110011; }

    function void display();
        $display("Instr: %h (rs1=%d, rs2=%d, rd=%d)", instr, rs1, rs2, rd);
    endfunction
endclass

```

### 2. Complete Layered Testbench Structure

```systemverilog
module tb_top;
    logic clk = 0;
    logic rst_n = 0;

    // Clock Generation
    always #5 clk = ~clk;

    // Interface Instantiation
    riscv_bus_if bus (clk, rst_n);

    // DUT Instantiation
    cpu_core dut (.bus(bus.master));

    // Stimulus Task
    initial begin
        riscv_transaction tx;
        tx = new();

        // System Reset
        #20 rst_n = 1;

        // Randomize & Drive
        repeat (5) begin
            if (tx.randomize()) begin
                tx.display();
            end
            @(posedge clk);
        end
        $finish;
    end
endmodule

```

---

## Sequential Circuits & FSM Design

A standard SystemVerilog state machine uses `enum` typed variables and separate procedural blocks for clear state transitions and output evaluations.

```systemverilog
module riscv_fsm (
    input  logic clk, rst_n,
    input  logic valid_instr,
    output logic fetch_enable, decode_enable
);
    typedef enum logic [1:0] {
        ST_RESET,
        ST_FETCH,
        ST_DECODE
    } state_t;

    state_t current_state, next_state;

    // State Register (Sequential)
    always_ff @(posedge clk or negedge rst_n) begin
        if (!rst_n) current_state <= ST_RESET;
        else        current_state <= next_state;
    end

    // Next State Logic (Combinational)
    always_comb begin
        next_state = current_state;
        case (current_state)
            ST_RESET:  next_state = ST_FETCH;
            ST_FETCH:  if (valid_instr) next_state = ST_DECODE;
            ST_DECODE: next_state = ST_FETCH;
            default:   next_state = ST_RESET;
        endcase
    end

    // Output Logic (Data-flow)
    assign fetch_enable  = (current_state == ST_FETCH);
    assign decode_enable = (current_state == ST_DECODE);
endmodule

```

---

## Logic Synthesis Guidelines

To ensure SystemVerilog code synthesizes cleanly to target FPGA/ASIC standard cells, follow standard synthesis rules.

### Synthesizable Rules

* Use `always_comb` for combinational logic and `always_ff` for sequential registers.
* Ensure all variables updated inside an `always_comb` block are assigned a default value at the beginning of the block to prevent latch inference.
* Always use non-blocking assignments (`<=`) in sequential blocks (`always_ff`) and blocking assignments (`=`) in combinational blocks (`always_comb`).
* Declare static array bounds inside loops; dynamic arrays, queues, and associative arrays are **not synthesizable**.

### Non-Synthesizable Constructs (Simulation Only)

* Delays (`#10`).
* Event waits (`wait`, `@`).
* Class instantiation, objects, and dynamically allocated memory (`new`).
* System tasks (`$display`, `$finish`, `$time`).
* Initial blocks (`initial` - except for inferred FPGA ROM initializations).