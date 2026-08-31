
The LLVM New Pass Manager (NPM)—introduced to replace the legacy `llvm::Pass` infrastructure—is a high-performance, template-driven framework for running transformations and analyses over LLVM IR.

## 1. Architectural Evolution: Legacy vs. New Pass Manager

The Legacy Pass Manager relied heavily on C++ runtime polymorphism (virtual inheritance). NPM switches to **concept-based type erasure** and the **Curiously Recurring Template Pattern (CRTP)**.

```
+-------------------------------------------------------------------------+
|                      LEGACY PASS MANAGER                               |
| - Class Inheritance: class MyPass : public FunctionPass                 |
| - Virtual Dispatch: vtable lookups for runOnFunction()                  |
| - Implicit Analysis: getAnalysis<DominatorTree>() declared in setup     |
| - Rigid IR Boundaries: Difficult to mix Function & Module passes cleanly|
+-------------------------------------------------------------------------+
                                    │
                                    ▼
+-------------------------------------------------------------------------+
|                      NEW PASS MANAGER (NPM)                             |
| - Generic Mixins: struct MyPass : PassInfoMixin<MyPass>                 |
| - Static Dispatch: Zero runtime vtable overhead                         |
| - Explicit Analysis: AnalysisManager passed directly to run()           |
| - Adaptors: Type-safe nesting across Module, CGSCC, Function, Loop      |
+-------------------------------------------------------------------------+

```

### Why NPM is Superior

1. **Performance:** Eliminates virtual function call overhead using static polymorphism.
2. **Explicit Analysis Tracking:** Passes explicitly declare which analysis results they preserve via `PreservedAnalyses`, avoiding redundant analysis recalculations.
3. **Thread-Safety & Parallelism:** Explicitly separated IR-unit boundaries enable multi-threaded pass execution across distinct functions/modules.
4. **Flexible Pipeline Composition:** Custom pipeline strings (e.g., `default<O2>`, `module(function(instcombine,dce))`) can be parsed and constructed at runtime.

---

## 2. Core Abstractions & Interfaces

### A. The CRTP Base: `PassInfoMixin<T>`

Every NPM pass inherits from `PassInfoMixin<PassT>` using CRTP:

```cpp
struct MemACPass : PassInfoMixin<MemACPass> { ... };

```

* **Purpose:** Provides a static compile-time reflection interface.
* **Key Functionality:** It automatically injects a `static StringRef name()` method derived from the class type name, eliminating the need to manually implement pass name getters.

---

### B. The `run()` Method Signature

The core entry point for any transform pass is its `run()` method:

```cpp
PreservedAnalyses run(IRUnitT &IR, AnalysisManagerT &AM);

```

* **`IRUnitT &IR`:** The specific LLVM IR construct the pass operates on (`Module`, `LazyCallGraph::SCC`, `Function`, or `Loop`).
* **`AnalysisManagerT &AM`:** The corresponding analysis manager (`ModuleAnalysisManager`, `FunctionAnalysisManager`, etc.) used to query cached analysis results lazily.

---

### C. `PreservedAnalyses` (Dependency & Invalidation Management)

NPM uses `PreservedAnalyses` return values to inform the pass infrastructure which cached analyses are still valid after IR modifications:

```
                      ┌─────────────────────────┐
                      │    Pass Execution       │
                      └────────────┬────────────┘
                                   │
              Did the pass modify the underlying IR?
                                   │
                  ┌────────────────┴────────────────┐
                 YES                                NO
                  │                                 │
        Which analyses stay valid?                  │
        ┌─────────┴──────────┐                      │
       ALL                  SOME                    │
        │                    │                      │
        ▼                    ▼                      ▼
PreservedAnalyses::   PA.preserve<...>()     PreservedAnalyses::
     none()           PA.preserveSet<...>()        all()

```

```cpp
// 1. No IR changes (Read-only analysis or reporting pass)
return PreservedAnalyses::all();

// 2. Heavy IR changes (Invalidates all cached analyses)
return PreservedAnalyses::none();

// 3. Selective IR changes (e.g., modified instructions, but Control Flow Graph remains untouched)
PreservedAnalyses PA = PreservedAnalyses::none();
PA.preserve<DominatorTreeAnalysis>(); // Keeps Dominator Tree
PA.preserveSet<CFGAnalyses>();        // Keeps LoopInfo, BranchProbability, etc.
return PA;

```

---

### D. The `isRequired()` Static Guard

```cpp
static bool isRequired() { return true; }
```

By default, Clang decorates functions with the `optnone` attribute when compiled with `-O0`. The Pass Manager skips non-essential passes on `optnone` functions unless this method returns `true`.

---

## 3. The IR Unit Hierarchy & Pass Adaptors

NPM enforces strict isolation between different IR granularities using dedicated Pass Managers and Adaptors:

| IR Unit | Pass Manager Type | Analysis Manager Type |
| --- | --- | --- |
| **`Module`** | `ModulePassManager` | `ModuleAnalysisManager` |
| **`CGSCC`** (Call Graph) | `CGSCCPassManager` | `CGSCCAnalysisManager` |
| **`Function`** | `FunctionPassManager` | `FunctionAnalysisManager` |
| **`Loop`** | `LoopPassManager` | `LoopAnalysisManager` |

### Adaptor Pattern

You cannot add a `FunctionPass` directly to a `ModulePassManager`. You must wrap it using an **Adaptor**:

```cpp
ModulePassManager MPM;
FunctionPassManager FPM;

// Add a function pass to a FunctionPassManager
FPM.addPass(MemACPass());

// Adapt FunctionPassManager to fit into a ModulePassManager
MPM.addPass(createModuleToFunctionPassAdaptor(std::move(FPM)));

```

---

## 4. Line-by-Line Code Breakdown of `MemACPass`

Below is an analysis of your custom plugin example:

### Part 1: Pass Logic (`MemACPass`)

```cpp
struct MemACPass : PassInfoMixin<MemACPass> {

    PreservedAnalyses run(Function &F, FunctionAnalysisManager &) {
        // Skip external function declarations (they lack basic blocks/body)
        if (F.isDeclaration()) {
            return PreservedAnalyses::all();
        }

        int total_loads = 0;
        int total_stores = 0;

        outs() << "Function: " << F.getName() << "\n";

        // Traverse Basic Blocks within the Function
        for (auto &BB : F) {
            // Traverse Instructions within each Basic Block
            for (Instruction &I : BB) {
                if (I.getOpcode() == Instruction::Load) {
                    total_loads += 1;
                } else if (I.getOpcode() == Instruction::Store) {
                    total_stores += 1;
                }
            }
        }

        outs() << "\tTotal: " << total_loads << " loads, " 
               << total_stores << " stores\n\n";

        // Read-only pass: preserve all analysis caches
        return PreservedAnalyses::all();
    }

    // Force execution even under -O0 (optnone)
    static bool isRequired() { return true; }
};

```

---

### Part 2: Plugin Registration Entry Point

NPM loads dynamic plugins via a C-linkage symbol called `llvmGetPassPluginInfo`:

```cpp
extern "C" LLVM_ATTRIBUTE_WEAK ::llvm::PassPluginLibraryInfo llvmGetPassPluginInfo() {
    return {
        LLVM_PLUGIN_API_VERSION, // API compatibility version check
        PLUGIN_NAME,             // Plugin identifier ("mem-ac-pass")
        LLVM_VERSION_STRING,     // LLVM source version matching
        [](PassBuilder &PB) {    // Callback to register hooks in PassBuilder
            
            /* -------------------------------------------------------------
             * Hook 1: Command-line pipeline parsing (-passes="mem-ac-pass")
             * ------------------------------------------------------------- */
            PB.registerPipelineParsingCallback(
                [](StringRef Name, FunctionPassManager &FPM,
                   ArrayRef<PassBuilder::PipelineElement>) {
                    if (Name == PLUGIN_NAME) {
                        FPM.addPass(MemACPass()); // Add pass to pipeline
                        return true;             // Handled
                    }
                    return false;                // Not target pass name
                });

            /* -------------------------------------------------------------
             * Hook 2: Automatic pipeline injection (Pipeline Extension Point)
             * ------------------------------------------------------------- */
            PB.registerPipelineStartEPCallback(
                [](ModulePassManager &MPM, OptimizationLevel Level) {
                    // Automatically run at the start of default pipelines (-O1, -O2, -O3)
                    MPM.addPass(createModuleToFunctionPassAdaptor(MemACPass()));
                });
        }
    };
}

```

---

## 5. Pipeline Extension Points (EPs)

The `PassBuilder` API provides several insertion hooks to inject custom passes into standard LLVM optimization pipelines (`-O1`, `-O2`, `-O3`, `-Os`):

```
          [ Standard Clang / Opt Optimization Pipeline ]
                                │
  registerPipelineStartEPCallback()        <-- Early instrumentation
                                │
                   [ Module / Function Passes ]
                                │
 registerVectorizationEPCallback()         <-- Before Vectorizer runs
                                │
    registerOptimizerLastEPCallback()      <-- Late cleanups before Codegen

```

---

## 6. Building and Running the Pass Plugin

### Step 1: Compilation (Shared Object)

Compile the C++ code into a dynamic plugin library (`.so` on Linux, `.dylib` on macOS):

```bash
clang++ -shared -fPIC `llvm-config --cxxflags` MemACPass.cpp \
  -o libMemACPass.so `llvm-config --ldflags`

```

### Step 2: Direct Execution via `opt`

Run the pass explicitly using the `-passes` command-line flag:

```bash
opt -load-pass-plugin=./libMemACPass.so \
    -passes="mem-ac-pass" \
    -disable-output target_code.ll

```

### Step 3: Injection into `clang` Engine

Inject the pass into normal compilation workflows using `-fpass-plugin`:

```bash
clang -fpass-plugin=./libMemACPass.so -O2 target_code.c -c -o target_code.o

```

---

## 7. Analysis Pass Integration (Querying Cached Data)

To fetch an existing analysis (such as the `DominatorTree`) inside your pass:

```cpp
#include "llvm/IR/Dominators.h"

PreservedAnalyses run(Function &F, FunctionAnalysisManager &FAM) {
    // Lazy query: Computes DominatorTree if not cached, or returns cached copy
    DominatorTree &DT = FAM.getResult<DominatorTreeAnalysis>(F);

    for (BasicBlock &BB : F) {
        if (DT.isReachableFromEntry(&BB)) {
            // ...
        }
    }
    return PreservedAnalyses::all();
}

```

---

## Phase 3 Summary Table

| API Construct | Scope | Responsibilities |
| --- | --- | --- |
| `PassInfoMixin<T>` | Pass Definition | Provides static metadata and name reflection via CRTP |
| `PreservedAnalyses` | Return Type | Manages invalidation of cached analysis results |
| `FunctionAnalysisManager` | Argument | Lazily evaluates and caches function-level IR analyses |
| `PassBuilder` | Plugin Registration | Registers CLI pipeline strings and extension point hooks |
| `createModuleToFunctionPassAdaptor` | Pass Adaptor | Wraps function passes so they run inside a module pipeline |

---