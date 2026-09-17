
Day to day life? Experience?

AI, usage internally, ubuntu/core, thoughts/productivity gains, personal experience

Ubuntu Core lighter, whats the target? Hardware Class, industry trends, immutable and secure?

Learning GO? Kernel Development? Any particular skill?

Future track of this application.





OS Requirement
Processes and Memory
Syscalls
IO and FD
cat example
shell example
Pipes
Filesystem
Page Tables
Walk example
sbrk example
exec impl example
Traps - Kernel vs Userspace
Syscall Mechanisms - Kernel vs Userspace
Device Interrupts





Embedded primitives
Kernel equivalents
DTS
Schedular algorithms
POSIX
Monolithic and other
per process page table
kernel mapped


- **CFS (Completely Fair Scheduler)** — Linux's default scheduler from 2.6.23 to fairly recently; tracks each task's "virtual runtime" and always picks the task with the least vruntime, stored in a red-black tree for O(log n) lookup — the goal is to approximate an "ideal" processor that gives every task an equal fair share.
- **EEVDF (Earliest Eligible Virtual Deadline First)** — replaced CFS as Linux's default scheduler (merged in 6.6); refines the same fairness idea but bounds _how late_ a task can run, giving better latency guarantees, especially useful for interactive/mixed workloads.