# Thoughts
Question -> Think -> Ask Clarification -> Think -> More back and forth -> maybe right it down -> Answer
# Words
autonomous, structured, agency
# Questions
## Me

"Tell me about yourself."
"Walk me through your resume / a project you're proud of."
"Tell me about a time you faced ambiguity or incomplete information."
"Tell me about a mistake or failure, and what you learned."
"Tell me about a disagreement or difficult collaboration."
"Why Canonical / why this team?"
"Compensation thoughts?"
"School Grades"
## Interviewer

"Next steps"
"What does the team's structure look like for someone joining at my level — who would I be working most closely with day to day?"
# Ubuntu Core
## Arch

- Packaging & OS Management: Snap is both the package format (compressed, read-only SquashFS images) and the runtime daemon (snapd) that manages installation, updates, and confinement.
- Ubuntu Core Infrastructure: The entire OS is composed of snaps. The Kernel, Gadget (bootloader/hardware layout), and Base (root runtime libraries) snaps exist strictly to provide atomic, transactional upgrades with automatic A/B rollbacks.
- Filesystem & Dependency Isolation (Namespaces): Application snaps run inside a private Mount Namespace, presenting a custom / root filesystem populated with the exact libraries and dependencies they need.
- Resource Management (cgroups v2): snapd hooks into systemd slices to cap and throttle CPU, RAM, and Disk I/O usage per application.
- Path & Device Access (AppArmor): Provides path-based Mandatory Access Control to restrict which files, DBus channels, network interfaces, or hardware nodes (/dev/) an application can touch.
- Syscall Filtering (Seccomp): Uses Berkeley Packet Filters (BPF) at the kernel level to block unauthorized or dangerous system calls from being executed by the application.

![[Screenshot from 2026-09-22 10-39-18.png]]

## Recent Developments

- Massive OTA Update Optimization (Snap Deltas)
- Shift to Chisel & Precise Provenance
- Cyber Resilience Act (CRA) Compliance & 15-Year LTS
- Hardware-Rooted Security Architecture
- Lean Base Snaps & Modular "Components"

## Future Roadmap

- Attested Edge AI Workloads
- Ubuntu Core Desktop
- Canonical Observability Stack (COS) Integration

# Kernel 

**1. OS** — Software layer that multiplexes hardware across programs while isolating them from each other.
**2. Process & Memory** — A running program with its own virtual address space (text, data, heap, stack).
- fork() — duplicates a process (copy-on-write in Linux)
- exec() — replaces current process image with a new program
**3. Syscalls** — Controlled trap from user mode into kernel mode to request privileged operations.
**4. I/O** — Kernel-mediated access to devices instead of direct hardware access.
- Blocking — caller sleeps until done
- Interrupt-driven/async — caller continues, notified later
**5. FD (File Descriptor)** — Per-process integer index pointing to an open file entry.
**6. Filesystem** — Abstraction mapping names/directories to data via inodes, unified across disk types by the VFS layer.
**7. Paging** — Virtual memory split into fixed-size pages mapped to physical frames via page tables, translated by the MMU (cached in the TLB).
**8. Traps & Exceptions** — Any forced switch from user to kernel mode.
- Syscall — deliberate request
- Exception — fault from the running instruction (page fault, divide-by-zero)
- Interrupt — external hardware event
**9. Interrupts** — Async hardware signals handled in two stages in Linux.
- Top half (ISR) — fast, minimal, runs immediately
- Bottom half (softirq/tasklet/workqueue) — deferred heavier work
**10. Schedulers** — Decides which task runs next on the CPU.
- Round robin — fixed slice, cycle through tasks
- Fixed-priority preemptive — highest priority ready task always runs (RTOS default)
- CFS — least virtual-runtime task picked, old Linux default
- EEVDF — current Linux default, bounds worst-case latency
- SCHED_FIFO/SCHED_RR — Linux real-time classes, above CFS/EEVDF

# Embedded 

- **Mutex** — Ownership-based lock; only the locking thread can unlock it.
- **Semaphore (counting)** — Counter with wait/signal, manages access to N identical resources.
- **Binary semaphore** — Semaphore capped at 0/1, used for signaling (no ownership, unlike mutex).
- **Spinlock** — Busy-waits instead of sleeping; used for very short critical sections or inside ISRs.
- **Condition variable** — Lets a thread sleep until signaled that a condition changed; always paired with a mutex.
- **Message queue** — FIFO of fixed-size messages for passing data between tasks.
- **Mailbox** — Like a message queue but holds a single item/pointer at a time.
- **Event flags/group** — Bitmask a task waits on until a combination of bits is set.
- **Critical section** — Code block run atomically, usually by disabling interrupts/scheduling.
- **ISR (Interrupt Service Routine)** — Handler function that runs when hardware raises an interrupt.
- **Watchdog timer** — Hardware timer that resets the system if not periodically "petted," catching hangs.
- **DMA** — Lets a peripheral move data to/from memory without per-byte CPU involvement.
- **Ring/circular buffer** — Fixed-size buffer with wrap-around pointers, common for ISR-to-task handoff.
- **TCB (Task Control Block)** — Struct holding a task's saved registers, stack pointer, priority, state.
- **Priority inversion** — Low-priority task holds a resource a high-priority task needs, stalled further by a medium-priority task preempting the low one.
- **Priority inheritance** — Fix for priority inversion: low-priority task temporarily inherits the blocked task's priority.
- **Tick / tickless idle** — Periodic timer interrupt driving the scheduler; tickless suppresses it when idle to save power.
- **Software timer** — Kernel-managed timer that fires a callback after a duration, without needing dedicated hardware timers.