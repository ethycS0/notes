# Implementation Varients

## Events

- When the processor executes a memory operation.
- When its data-flow integrity (DFI) check completes.
- When the processor allows that operation's architectural effects to become visible.

## Co-Processor

### Advantages

1. Simpler separation of responsibilities
2. Lower timing-closure risk
3. Opportunities for asynchronous execution
4. Easier independent experimentation

### Disadvantages

1. A detected violation may already have caused architectural effects
2. The checker must keep up with committed memory traffic
3. Correctness requires strict metadata ordering
4. Errors are generally imprecise
5. Larger system-level verification burden than it initially appears

### FU

### Advantages

1. Precise verification before retirement
2. Overlap with memory execution
3. Better use of speculative knowledge
4. Recovery can reuse existing core mechanisms

### Disadvantages

1. Increased pressure on retirement
2. More complicated speculative state (Store messing with RDTable)
3. Potentially expensive graduation-list modifications
4. Physical timing risks


# Optimisations Slides

1. Post vs pre commit
2. Cache