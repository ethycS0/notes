Andersen's Points-To Analysis is an inclusion-based, flow-insensitive, context-insensitive static analysis algorithm used within [[flow-insensitive_context-insensitive_inter-procedural_analysis|FICIIPA]] to determine the complete set of memory locations (points-to sets) that each pointer variable can reference during program execution. It provides the foundation for resolving memory aliasing in Data-Flow Integrity (DFI) by computing which memory targets are reachable via pointer dereferences.
### Constraint Representation and Rule Types
The algorithm models pointers and variables as sets of abstract memory locations, where $pts(p)$ denotes the points-to set of pointer variable $p$. Statements in the High-Level Intermediate Representation (HIR) involving memory addresses or assignments are converted into four standardized constraint rules:

* Address-of Assignment (`p = &a`): Directly populates a pointer set with a target variable. Represented as the membership constraint $a \in pts(p)$.
* Pointer Copy (`p = q`): Specifies that pointer $p$ receives all target memory locations held by pointer $q$. Represented as the inclusion/subset constraint $pts(p) \supseteq pts(q)$.
* Load Dereference (`p = *q`): Specifies that $p$ takes targets from the memory locations pointed to by $q$. For every target variable $v \in pts(q)$, it generates the conditional constraint $pts(p) \supseteq pts(v)$.
* Store Dereference (`*p = q`): Specifies that values in $q$ are stored into locations pointed to by $p$. For every target variable $v \in pts(p)$, it generates the conditional constraint $pts(v) \supseteq pts(q)$.
### Fixed-Point Solving Engine
Andersen's analysis solves pointer targets through monotonic set growth rather than set reduction:

1. Initialization: Points-to sets for all pointers start empty, except for initial target locations explicitly seeded by address-of statements ($a \in pts(p)$).
2. Propagation Pass: The solver repeatedly evaluates the global collection of subset constraints collected across all source files. Whenever a constraint $pts(p) \supseteq pts(q)$ is evaluated, any targets present in $pts(q)$ that are missing from $pts(p)$ are copied into $pts(p)$.
3. Fixed-Point Convergence: The propagation pass loops continuously over all program constraints until an entire cycle produces zero new additions to any pointer's points-to set. At this fixed point, the points-to sets have expanded to hold all possible targets for every pointer variable in the codebase.
### Field and Context Abstractions
To remain computationally tractable across large programs, Andersen's analysis in DFI applies specific structural abstractions:

* Field-Insensitivity: Structure, class, and union fields are aggregated into a single abstract memory object. Accesses to `ptr->fieldA` and `ptr->fieldB` modify and read from the same points-to set.
* Context-Insensitivity: Execution paths through functions are merged across call sites. A function's internal pointer operations contribute to the same global subset constraints regardless of which caller invoked it.
### Integration with Reaching Definition Construction
Once fixed-point iteration completes, the solved points-to sets $pts(p)$ are consumed by [[FICIIPA]] to construct static reaching definition sets. When a write instruction targets a pointer dereference `*p`, DFI inspects $pts(p)$ and tags that write instruction ID as a valid reaching definition for every variable $v$ contained within $pts(p)$. This ensures that all indirect pointer modifications are safely accounted for when generating runtime `CHECKDEF` verification sets.