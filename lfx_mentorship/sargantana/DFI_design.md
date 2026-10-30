# DFI datapath integration

| Decision          | Current approach                                                                                                                                                                         |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Instruction order | `custom0` prefix immediately before its protected memory operation.                                                                                                                      |
| Encoding          | U-type payload: 16-bit ID and 2-bit L/S/A class. No register reads or writes.                                                                                                            |
| Request creation  | Send defID, L/S/A and GL index when the prefix actually allocates into GL.                                                                                                               |
| Association       | Bind the next allocated memory instruction to the prefix; validate its operation class.                                                                                                  |
| Load protection   | Hold the prefix until the DFI check passes. On mismatch, fault the prefix. This blocks retirement; speculative load execution is still possible.                                         |
| Store protection  | Release the prefix after valid PA capture and reservation of tracking capacity. Update RDTable when the associated store successfully commits.                                           |
| Retirement        | Prefix and memory operation retire separately. Keep store tracking alive after prefix retirement.                                                                                        |
| Flushes           | Discard squashed associations and reject stale responses after GL index reuse. Keep committed metadata updates until completed.                                                          |
| Interrupts        | Defer interrupt attachment across the protected memory instruction. Interrupts may be taken at the prefix or after the pair. Synchronous faults retain a separate pair-restart protocol. |
