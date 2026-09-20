
`vset_queue.sv` stores vector configuration states produced by [[vset_module|VSET Module]]. It is a circular buffer that allows multiple VSET states to exist while instructions are speculative. Each entry stores the vector state associated with one VSET, and the queue updates its pointers when a VSET is inserted, committed, or discarded during recovery.

The queue always retains at least one entry. After reset, entry `0` represents the initial vector state, `tail` points to entry `1`, and `num` is `1`. New VSET states are written at `tail`, while `index_last_vset` identifies the most recently inserted VSET whose state is exposed to the rest of the design.

The queue supports three types of state movement:

| Event                        | Effect                                                 |
| ---------------------------- | ------------------------------------------------------ |
| `new_vset_i`                 | Add a new speculative VSET                             |
| `commited_vset_i`            | Advance the committed VSET state                       |
| `recover_last_committed_i`   | Remove speculative VSET state after a commit exception |
| `recover_last_misspredict_i` | Remove VSET state created after a branch misprediction |

## Queue Entry

Each queue element is a `vset_entry_t`.

| Field             | Meaning                         |
| ----------------- | ------------------------------- |
| `vl`              | Vector length for this VSET     |
| `vtype`           | Vector configuration            |
| `vnarrow_wide_en` | Narrow or wide operation enable |
| `vlmax`           | Maximum vector length           |

The queue therefore stores enough information to reconstruct the vector configuration associated with a VSET.

## Queue Pointers

| Signal                      | Purpose                                   |
| --------------------------- | ----------------------------------------- |
| `last_committed_vset_index` | Index of the most recently committed VSET |
| `index_last_vset`           | Index of the newest VSET currently stored |
| `tail`                      | Location where the next VSET is written   |
| `first_not_committed_index` | First VSET after the committed state      |
| `num`                       | Number of active entries                  |

`first_not_committed_index` is combinationally calculated as:

```systemverilog
last_committed_vset_index + 1
```

`vset_index_o` is driven from `index_last_vset`, so it identifies the newest VSET entry.

## Queue Control Flags

The queue derives two main control signals.

| Signal                  | Condition                                        | Example                                      |
| ----------------------- | ------------------------------------------------ | -------------------------------------------- |
| `write_new_vset_enable` | `new_vset_i` and queue not full                  | [[decoder\|Decoder]] accepts a new `VSETVLI` |
| `next_committed_enable` | `commited_vset_i` and more than one entry exists | ROB commits the oldest speculative VSET      |

`write_new_vset_enable` prevents a new entry from being written when `num == VSET_QUEUE_NUM_ENTRIES`.

`next_committed_enable` requires `num > 1`. This prevents the queue from removing its final entry, which represents the currently retained vector state.

## Circular Buffer

The queue uses `tail` as the write pointer.

When `write_new_vset_enable` is asserted:

```systemverilog
vset_buffer[tail].vl = vl_i
vset_buffer[tail].vtype = vtype_i
vset_buffer[tail].vnarrow_wide_en = vnarrow_wide_en_i
vset_buffer[tail].vlmax = vlmax_i
```

After the write, `tail` advances using `trunc_iq_ptr_sum`, so the pointer wraps around the queue size.

The queue therefore behaves as a circular buffer rather than a shifting FIFO.

## Queue Entry Count

`num` tracks the number of active entries.

Normal operation updates it according to:

| Event               | Change to `num` |
| ------------------- | --------------: |
| New VSET only       |            `+1` |
| Commit only         |            `-1` |
| Neither             |             `0` |
| New VSET and commit |             `0` |

The implementation uses `trunc_iq_num_sum` to keep the counter width appropriate for the queue.

## Reset State

On reset:

| State                       | Value |
| --------------------------- | ----- |
| `last_committed_vset_index` | `0`   |
| `index_last_vset`           | `0`   |
| `tail`                      | `1`   |
| `num`                       | `1`   |

Every buffer entry is also initialized.

The stored initial entry has:

```text
VL = 0
VTYPE = illegal state
vnarrow_wide_en = 0
VLMAX = 0
```

The queue therefore starts with one retained vector state rather than an empty buffer.

## Adding a VSET

When `write_new_vset_enable` is asserted, the new state from [[vset_module|VSET Module]] is written into `vset_buffer[tail]`.

The state stored for the VSET is:

| Input               | Stored field                        |
| ------------------- | ----------------------------------- |
| `vl_i`              | `vset_buffer[tail].vl`              |
| `vtype_i`           | `vset_buffer[tail].vtype`           |
| `vnarrow_wide_en_i` | `vset_buffer[tail].vnarrow_wide_en` |
| `vlmax_i`           | `vset_buffer[tail].vlmax`           |

At the same time:

```text
index_last_vset
tail
num
```

are updated to include the new entry.

## Committing a VSET

`commited_vset_i` advances the committed position through the queue.

The committed index is stored in:

```systemverilog
last_committed_vset_index
```

When a VSET commits, this pointer advances by one.

The queue does not remove the final retained entry because:

```systemverilog
next_committed_enable = commited_vset_i & (num > 1);
```

Therefore the queue always keeps one vector state available.

## Recovering After a Commit Exception

`recover_last_committed_i` discards all speculative VSET state and returns the queue to the most recently committed state.

The important updates are:

| State             | Recovery value                  |
| ----------------- | ------------------------------- |
| `index_last_vset` | `last_committed_vset_index`     |
| `tail`            | `last_committed_vset_index + 1` |
| `num`             | `1`                             |

This leaves only the committed vector configuration in the queue.

## Recovering After a Branch Misprediction

`recover_last_misspredict_i` uses:

```systemverilog
vset_index_misspredict_i
```

to identify where the speculative VSET history must end.

The queue calculates:

```text
num_in_misspred
```

from the last committed index and the supplied misprediction index.

The recovery then sets:

| State             | Recovery value                                 |
| ----------------- | ---------------------------------------------- |
| `index_last_vset` | `vset_index_misspredict_i`                     |
| `tail`            | `vset_index_misspredict_i + 1`                 |
| `num`             | Number of entries remaining after the rollback |

If a commit and misprediction recovery happen together, `last_committed_vset_index` is also advanced before the final queue state is calculated.

## Misprediction Entry Count

The queue calculates:

```systemverilog
num_in_misspred_prev
num_in_misspred
```

to determine how many VSET entries remain after a misprediction.

This calculation accounts for the circular buffer wrapping between:

```text
last_committed_vset_index
vset_index_misspredict_i
```

The helper functions `trunc_iq_num_sum` and `trunc_iq_ptr_sum` keep the arithmetic within the required counter and pointer widths.

## Current Vector State

The outputs are read directly from:

```systemverilog
vset_buffer[index_last_vset]
```

| Output              | Stored value                                   |
| ------------------- | ---------------------------------------------- |
| `vl_o`              | `vset_buffer[index_last_vset].vl`              |
| `vlmax_o`           | `vset_buffer[index_last_vset].vlmax`           |
| `vnarrow_wide_en_o` | `vset_buffer[index_last_vset].vnarrow_wide_en` |

The remaining vector state is extracted from the stored `vtype`.

| Output    | `vtype` field |
| --------- | ------------- |
| `vill_o`  | `[8]`         |
| `sew_o`   | `[4:3]`       |
| `vta_o`   | `[6]`         |
| `vma_o`   | `[7]`         |
| `vlmul_o` | `[2:0]`       |

## Previous VTYPE

`prev_vtype_o` is read from:

```systemverilog
vset_buffer[first_not_committed_index]
```

where:

```systemverilog
first_not_committed_index =
    last_committed_vset_index + 1
```

Therefore `prev_vtype_o` represents the VTYPE belonging to the first VSET that has not yet committed.

This is different from `vtype` of the newest VSET. It specifically identifies the state immediately after the committed boundary.

## Queue Status

`full_o` is asserted when:

```systemverilog
num == VSET_QUEUE_NUM_ENTRIES
```

or reset is active.

| `full_o` | Meaning                          |
| -------: | -------------------------------- |
|      `0` | Queue can accept another VSET    |
|      `1` | Queue is full or reset is active |

`full_o` is exposed through [[vset_module|VSET Module]] as `full_vset_queue_o`.

## VSET State Flow

| Stage     | Module          | Responsibility                                |
| --------- | --------------- | --------------------------------------------- |
| Decode    | [[decoder]]     | Identify `VSETVL`, `VSETVLI`, or `VSETIVLI`   |
| Calculate | [[vset_module]] | Calculate `VL`, `VLMAX`, and validate `VTYPE` |
| Store     | `vset_queue.sv` | Store the resulting vector state              |
| Commit    | `vset_queue.sv` | Advance the committed VSET                    |
| Recovery  | `vset_queue.sv` | Remove speculative VSET state after recovery  |
| Consume   | Vector pipeline | Use the state exposed by the queue            |

## VSET Queue Working

The queue is not simply storing the latest `VL`.

It keeps a history of VSET states so that speculative vector configuration changes can coexist with committed state. A new VSET creates another queue entry. A committed VSET advances the committed boundary. A recovery operation moves the queue back to the appropriate VSET entry and discards the speculative entries after it.

This makes the queue the state-tracking mechanism between the VSET calculation in [[vset_module|VSET Module]] and the vector execution state consumed by the rest of the processor.

## Key Signals

| Signal                       | Meaning                               |
| ---------------------------- | ------------------------------------- |
| `new_vset_i`                 | Insert a VSET state                   |
| `commited_vset_i`            | Commit the oldest uncommitted VSET    |
| `recover_last_committed_i`   | Roll back to committed state          |
| `recover_last_misspredict_i` | Roll back to a misprediction boundary |
| `vset_index_misspredict_i`   | Misprediction rollback index          |
| `vset_index_o`               | Newest stored VSET index              |
| `prev_vtype_o`               | First uncommitted VTYPE               |
| `full_o`                     | Queue full                            |
| `num`                        | Number of active VSET states          |

## Core Idea

`vset_queue.sv` is a circular buffer for speculative vector configuration.

Its main responsibility is maintaining the relationship between:

```text
committed vector state
speculative VSET states
```

while allowing new VSET states to be added, committed in order, or removed during exception and branch misprediction recovery.
