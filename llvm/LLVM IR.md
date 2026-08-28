
LLVM IR is a universal, strongly typed, RISC-like instruction set designed to represent code at an intermediate level. It bridges high-level source languages with low-level target hardware.

---

## 1. Core Properties & Forms

LLVM IR is built around three fundamental design principles:

* **Static Single Assignment (SSA):** Every register/variable is assigned **exactly once**. Existing values are immutable; modifications require creating a new virtual register. This eliminates complex alias analysis for local variables during optimizations.
* **Infinite Virtual Register Set:** Unlike physical CPUs with limited registers (e.g., 16 in x86-64), LLVM IR assumes an unlimited pool of temporary registers (`%0`, `%1`, `%a`, `%b`).
* **Strong Type System:** Every value and instruction parameter is explicitly typed (e.g., `i32`, `float`, `ptr`).

### The Three Equivalent Representations

LLVM IR exists in three isomorphic forms (losslessly convertible into one another):

| Form | File Format | Format Type | Primary Purpose |
| --- | --- | --- | --- |
| **In-Memory** | N/A (C++ Classes) | C++ Object Graph | Active manipulation inside the compiler compiler passes (`llvm::Module`, `llvm::Instruction`) |
| **On-Disk Bitcode** | `.bc` | Dense Binary | Space-efficient storage, fast parsing, link-time optimization (LTO) |
| **Human-Readable Text** | `.ll` | Text Assembly | Debugging, pass testing, manual inspection, learning |

---

## 2. The IR Hierarchy (Structural Building Blocks)

LLVM IR is structured as a top-down object ownership hierarchy:

```
Module (*.ll / *.bc)
  ├── Target Triple & Data Layout
  ├── Global Variables (@var)
  └── Functions (@func)
        └── Basic Blocks (%entry, %loop)
              └── Instructions (%1 = add i32 %a, %b)

```

* **`llvm::Module`:** Represents a single translation unit. Holds target architecture metadata, global variables, function prototypes, and function definitions.
* **`llvm::Function`:** Contains function signature, arguments, attributes, and a list of basic blocks.
* **`llvm::BasicBlock`:** A sequence of instructions executed linearly.
> **Rule:** Must have **exactly one entry** (the first instruction) and **exactly one exit point** at the end, called a **Terminator Instruction** (e.g., `ret`, `br`, `switch`).
* **`llvm::Instruction`:** An atomic operation (e.g., `add`, `load`, `store`, `phi`, `call`).

---

## 3. Key Syntax Rules & Concepts

* **Prefix Identifiers:**
* `@` = Global symbols (functions, global variables).
* `%` = Local symbols (virtual registers, basic block labels).


* **Types:**
* Integers: `i1` (bool), `i8` (byte), `i32`, `i64`.
* Floating-point: `half`, `float`, `double`.
* Pointers: `ptr` (opaque pointer type used in modern LLVM).
* Aggregate Types: Arrays `[10 x i32]`, Structs `{ i32, ptr }`.


* **Memory Model (`load` / `store`):**
* Registers cannot have their addresses taken. Local variables that require memory addresses (e.g., `&x` in C) are allocated on the stack using `alloca` and accessed via `load` and `store`.


* **The `phi` Node (SSA Merging):**
* Because variables cannot be re-assigned, `phi` instructions select a value based on which basic block control flow just came from.



---

## 4. Concrete Example: C to LLVM IR

### C Code

```c
int max(int a, int b) {
    if (a > b) return a;
    else return b;
}

```

### Equivalent Human-Readable LLVM IR (`.ll`)

```llvm
define i32 @max(i32 %a, i32 %b) {
entry:
  ; Compare %a and %b (signed greater-than) -> returns i1 (boolean)
  %cmp = icmp sgt i32 %a, %b
  
  ; Conditional branch: jump to %then if true, %else if false
  br i1 %cmp, label %then, label %else

then:
  ret i32 %a

else:
  ret i32 %b
}

```

---

## 5. Essential Tooling Cheat Sheet

The LLVM infrastructure provides CLI utilities to generate, disassemble, optimize, and execute IR files directly:

### Emitting LLVM IR from Clang

```bash
# Generate human-readable IR (.ll) from C code (unoptimized)
clang -S -emit-llvm -O0 -Xclang -disable-O0-optnone main.c -o main.ll

# Generate binary bitcode (.bc)
clang -c -emit-llvm main.c -o main.bc

```

### IR Conversion Tools

```bash
# Assemble human-readable text (.ll) into binary bitcode (.bc)
llvm-as main.ll -o main.bc

# Disassemble binary bitcode (.bc) back into text IR (.ll)
llvm-dis main.bc -o main.ll

```

### Pipeline Execution & Debugging Tools

```bash
# Run optimization passes on text IR using the New Pass Manager
opt -passes="mem2reg,instcombine,dce" main.ll -S -o main_opt.ll

# Compile LLVM IR directly to target assembly (e.g., x86-64 or ARM)
llc main.ll -o main.s

# Execute LLVM IR directly using LLVM's JIT compiler / interpreter
lli main.ll

```

---