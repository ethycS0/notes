# Sargantana DFI Policy

## 1. DFI Model

Data-Flow Integrity restricts the definitions from which a protected load may obtain data.

Compilation produces a conservative Reaching Definition Set (RDSet) for each protected load. This set includes all definitions that may legitimately supply the load under the analysis model, including effects of partial writes and read/modify/write operations.

At runtime:

- A successful protected store records its definition identifier in the Reaching Definition Table (RDTable).
- A protected load verifies that the recorded identifiers for its accessed data belong to its RDSet.
- A failed verification raises a DFI violation.

## 2. Identifiers and Policy Tables

Identifiers are 16 bits wide.

A protected store carries a **definition ID**. A protected load carries an **ID selecting its RDSet**. The operation class determines the interpretation of the identifier field.

| Structure   | Purpose                                                                        |
| ----------- | ------------------------------------------------------------------------------ |
| **RDTable** | Records the definition identifier associated with each protected data granule. |
| **RDSMap**  | Maps a protected load’s identifier to its RDSet.                               |
| **RDSets**  | Contain the permitted definition identifiers for protected loads.              |

Application software must not modify these structures outside the authorized mechanisms for loading policy and maintaining runtime labels.

> Identifier allocation, reserved identifiers, and initial RDTable values remain to be specified.

## 3. Metadata Granularity

One 16-bit RDTable label is associated with each aligned 32-bit data granule. RDTable storage overhead is 50% relative to protected data capacity.

Every granule covered by a protected access participates in DFI:

- A load verifies every accessed granule.
- A successful store assigns its definition ID to every affected granule.
- An aligned 64-bit access covers two granules.

Access width is determined by the memory instruction.

### Compiler contract

Independently tracked objects must not share a four-byte metadata granule. Fields or elements sharing a granule must be analyzed together.

The treatment of partial writes, packed objects, and externally fixed layouts remains to be finalized under this contract.

## 4. Protected Memory Operation

A DFI prefix and its immediately following memory instruction constitute **one architectural protected memory operation**.

The pair has:

- One architectural completion outcome.
- One fault and restart boundary, identified by the prefix PC.
- Joint readiness for graduation.
- Joint treatment during recovery.

Although the pair consists of two instruction encodings, its components do not complete independently at the architectural level.

The components may retire in different cycles, provided this exposes no intermediate architectural state and permits no interrupt or recovery boundary between them.

## 5. Prefix and Pairing Requirements

The DFI prefix:

- Supplies a 16-bit identifier.
- Identifies the operation class: load, store, or atomic.
- Reads and writes no architectural registers.
- Has no independent ordinary data-memory side effect.

The memory instruction must immediately follow the prefix in the sequential instruction stream, and its operation class must match the prefix.

For a 32-bit prefix:

```
memory instruction PC = prefix PC + 4
```

Neither component may commit independently before the pair’s completion outcome is resolved.

All control-flow transfers, including direct and indirect jumps and conditional branches, into a protected memory operation must target its DFI prefix. Entry directly into the associated memory instruction is prohibited.

Malformed-pair behavior will trap as illegal instruction (xcause 2).

### Encoding status

The prototype uses the `custom0` opcode space with U-format encoding containing the identifier and operation class.

## 6. Protected Loads

A protected load must pass DFI verification before the protected operation completes architecturally.

For every accessed granule, the checked definition identifier must correspond to the data observed by the load.

The operation succeeds only when:

- The memory access completes without an ordinary memory fault.
- Every required label belongs to the load’s RDSet.

A failed check prevents the protected operation and younger instructions from completing architecturally.

This is a pre-commit enforcement policy. Prevention of speculative side effects is not established by this policy. Architectural correctness is maintained.

## 7. Protected Stores

A protected store assigns its definition ID to every affected RDTable granule only upon successful architectural completion.

A store that faults or is discarded must not update the RDTable.

The prefix and store become eligible for graduation together, once the pair’s successful completion outcome is resolved. Address resolution alone does not establish successful completion.

Subsequent protected loads must observe labels consistent with the store data they observe.

## 8. Atomic Operations

The prefix encoding reserves an atomic operation class.

> Read-verification and write-labeling semantics for atomic operations, including LR/SC, remain to be specified. Atomic protection is not yet a completed part of the baseline policy.

## 9. Faults and Restart

Any fault associated with a protected memory operation is attributed to the pair’s prefix PC.

A faulting pair does not complete successfully. Returning to its exception PC restarts execution from the prefix.

### DFI violations

A failed DFI check raises a **Software Check Exception**, with cause **18 decimal (`0x12`)**.

- The exception PC identifies the prefix.
- The protected operation does not complete architecturally.
- Younger instructions do not complete architecturally.

> The `xtval` convention and DFI-violation recovery behavior remain to be specified.

### Ordinary memory faults

Translation, access, and other ordinary memory faults retain their original exception causes and required fault information.

- The exception PC identifies the prefix.
- Fault-address information identifies the faulting memory access.
- The pair reaches exception handling without requiring successful DFI verification.
- A retry begins at the prefix.

This exception-PC convention is part of the protected-operation architecture.

> Exception priority when multiple fault conditions are present remains to be specified.

## 10. Interrupts

An interrupt must not create an architectural boundary between the prefix and its associated memory instruction.

An interrupt may be taken:

- Before the protected operation begins architectural completion, with restart at the prefix.
- After the protected operation completes.

An interrupt deferred across the pair must remain pending.

## 11. Flushes and Recovery

An incomplete protected memory operation is discarded as a pair.

Recovery must not leave one component architecturally completed while discarding or independently restarting the other.

Discarded protected operations have no lasting architectural DFI effects:

- Discarded stores do not modify the RDTable.
- Discarded checks do not raise architectural DFI exceptions.
- Metadata updates required by successfully completed stores are preserved.

## 12. Unprotected Operations

Memory instructions without a DFI prefix remain permitted, supporting unanalyzed assembly and libraries.

Such instructions are ordinary memory operations and do not receive the protected-pair semantics defined above. Unprotected loads receive no DFI verification.

> The treatment of unprotected stores into protected data, and contracts for transitions between instrumented and uninstrumented code, remain to be specified.