![[Pasted image 20260828121327.png]]

## 1. The Core Conceptual Model: "Climbing the Mountain"

A compiler pipeline is best conceptualized as climbing up and down a mountain:

* **Ascent (Analysis):** The compiler takes low-level raw text and continuously elevates it to higher levels of abstraction where program semantics (meaning) become explicit.
* **Peak (Understanding):** At the highest point of abstraction, the compiler fully understands program semantics, scopes, and types.
* **Descent (Synthesis):** The compiler transforms the high-level semantic model back down through lower-level representations until it reaches raw machine instructions.

```
       [ Highest Semantic Insight ]
                 /   \
  Static Analysis     \  Middle End / IR Optimization
               /       \
        Parsing         \ Code Generation
             /           \
         Lexing           \ Machine / Byte Code
            |               |
     [ Source Text ]   [ Target Executable ]

```

---

## 2. Front End (Language-Specific Analysis)

The front end takes source code written by a human and analyzes its syntax and semantics.

### Phase A: Scanning (Lexical Analysis / Lexing)

* **Input:** A linear character stream (`var average = (min + max) / 2;`).
* **Output:** A flat sequence of **Tokens** representing meaningful syntactic units.
* **Mechanism:** Group raw characters into chunks while discarding irrelevant characters (whitespace, comments).
* **Token Types:**
* **Keywords:** `var`, `if`, `return`
* **Identifiers:** `average`, `min`, `max`
* **Operators & Punctuation:** `=`, `+`, `/`, `(`, `)`
* **Literals:** `2`, `"hello"`



### Phase B: Parsing (Syntax Analysis)

* **Input:** Sequence of tokens from the scanner.
* **Output:** An **Abstract Syntax Tree (AST)** or Parse Tree.
* **Mechanism:** Enforces the language's formal grammar rules, grouping tokens into nested tree structures that represent expressions and statements.
* **Error Handling:** The parser detects and reports syntax errors (e.g., missing closing parenthesis or unexpected operators).

### Phase C: Static Analysis (Semantic Analysis)

* **Input:** Raw AST.
* **Output:** Decorated AST / **Symbol Table**.
* **Key Tasks:**
1. **Name Resolution / Binding:** Maps variable names (identifiers) to their exact definitions across lexical scopes.
2. **Type Checking:** Verifies that operations are valid for the given data types (in statically typed languages) and reports type errors.


* **Data Storage:** Stores semantic context directly as node attributes on the AST or in a side lookup table called a **Symbol Table**.

---

## 3. Middle End (Optimization & Decoupling)

The middle end isolates source-language syntax from target-hardware details.

### Intermediate Representation (IR)

* **Concept:** A universal, machine-independent representation of code semantics.
* **The $N \times M$ Problem:** Without an IR, supporting $N$ source languages across $M$ CPU architectures requires building $N \times M$ complete compilers. With a shared IR, you only write $N$ front ends and $M$ back ends ($N + M$).
* **Common Forms:**
* **Static Single-Assignment (SSA):** Every variable is assigned exactly once (essential for LLVM).
* **Three-Address Code (TAC):** Quadruples/Triples of standard form `x = y op z`.
* **Control Flow Graph (CFG):** Basic blocks connected by directed edges representing jumps and branches.



### Optimization

* **Goal:** Transform the IR into an equivalent IR that executes faster or consumes fewer resources without altering program output.
* **Classic Optimization Passes:**
* **Constant Folding:** Evaluate constant expressions at compile time (e.g., `3.14 * 2` $\rightarrow$ `6.28`).
* **Dead Code Elimination (DCE):** Remove instructions whose results are never used.
* **Common Subexpression Elimination (CSE):** Avoid redundant recalculations of identical subexpressions.
* **Loop Invariant Code Motion (LICM):** Hoist calculations out of loops if their inputs don't change inside the loop.



---

## 4. Back End (Target Code Generation)

The back end translates optimized IR into target-specific executable forms.

### Code Generation & Register Allocation

* **Machine Code:** Generates binary native instructions tailored to a physical CPU architecture (x86-64, ARM64, RISC-V).
* **Instruction Selection:** Maps high-level IR operations to optimal target machine instructions.
* **Register Allocation:** Maps an unbounded number of virtual IR variables to a finite set of hardware registers, spilling excess variables to stack memory.

### Alternative Execution Strategies

* **Native Compilation:** Direct compilation to machine code. Maximum speed, minimal portability across architectures.
* **Virtual Machine Bytecode:** Target a portable, synthetic instruction set (e.g., JVM, WebAssembly, Python bytecode) evaluated by an interpreter or Virtual Machine.
* **Just-In-Time (JIT) Compilation:** Dynamic compilation where bytecode is translated to native machine code at runtime based on hot execution paths.

---

## Quick Summary Table

| Phase | Input | Output | Primary Responsibility |
| --- | --- | --- | --- |
| **Scanning** | Character Stream | Token Stream | Lexical analysis, removing comments/whitespace |
| **Parsing** | Token Stream | AST | Grammatical validation and structural tree building |
| **Static Analysis** | AST | Decorated AST / Symbol Table | Scope resolution, binding, and type checking |
| **IR Generation** | AST | Intermediate Representation | Converting language-specific AST into standard IR |
| **Optimization** | IR | Optimized IR | Improving execution efficiency and reducing code size |
| **Code Generation** | Optimized IR | Assembly / Machine / Bytecode | Target-specific instruction selection & register allocation |

---