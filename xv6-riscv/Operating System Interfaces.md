* **Kernel:** A special program that provides services to running programs via system calls, isolating processes and managing shared hardware resources.
* **Process:** An abstract unit of execution consisting of user-space memory (instructions, data, stack) and per-process state private to the kernel.
* **Syscalls:** The explicit interface through which user programs invoke services managed by the kernel.
* **User/Kernel Space:** The separation of execution environments where user space runs restricted application code and kernel space runs privileged management code.
* **Privilege:** Hardware-enforced protection levels that dictate whether executing code can access raw hardware, page tables, and control registers.
# Syscalls

| System call                             | Description                                                              |
| --------------------------------------- | ------------------------------------------------------------------------ |
| `int fork()`                            | Create a process, return child's PID.                                    |
| `int exit(int status)`                  | Terminate the current process; status reported to wait(). No return.     |
| `int wait(int *status)`                 | Wait for a child to exit; exit status in *status; returns child PID.     |
| `int kill(int pid)`                     | Terminate process PID. Returns 0, or -1 for error.                       |
| `int getpid()`                          | Return the current process's PID.                                        |
| `int sleep(int n)`                      | Pause for n clock ticks.                                                 |
| `int exec(char *file, char *argv[])`    | Load a file and execute it with arguments; only returns if error.        |
| `char *sbrk(int n)`                     | Grow process's memory by n bytes. Returns start of new memory.           |
| `int open(char *file, int flags)`       | Open a file; flags indicate read/write; returns an fd (file descriptor). |
| `int write(int fd, char *buf, int n)`   | Write n bytes from buf to file descriptor fd; returns n.                 |
| `int read(int fd, char *buf, int n)`    | Read n bytes into buf; returns number read; or 0 if end of file.         |
| `int close(int fd)`                     | Release open file fd.                                                    |
| `int dup(int fd)`                       | Return a new file descriptor referring to the same file as fd.           |
| `int pipe(int p[])`                     | Create a pipe, put read/write file descriptors in p[0] and p[1].         |
| `int chdir(char *dir)`                  | Change the current directory.                                            |
| `int mkdir(char *dir)`                  | Create a new directory.                                                  |
| `int mknod(char *file, int, int)`       | Create a device file.                                                    |
| `int fstat(int fd, struct stat *st)`    | Place info about an open file into *st.                                  |
| `int stat(char *file, struct stat *st)` | Place info about a named file into *st.                                  |
| `int link(char *file1, char *file2)`    | Create another name (file2) for the file file1.                          |
| `int unlink(char *file1)`               | Remove a file.                                                           |
# 1.1 Processes and Memory

- Time sharing: The kernel transparently switches CPU execution across available processes to provide the illusion of simultaneous execution.

- PID: Process Identifier; a unique positive integer assigned by the kernel to uniquely track each active process.

- Process vs Thread: A process is an isolated container holding a private address space and a thread of execution; xv6 does not support multi-threaded processes (one process contains exactly one thread).

### fork()

A system call where a process creates a duplicate of itself, yielding an identical memory image, file descriptor table, and register state.

- Memory Space / Registers: The child process gets a separate, isolated physical memory copy (and independent register values) that behaves completely isolated from the parent.

- COW: Copy-on-Write; an optimization (not present in basic xv6, but a common extension) where parent and child share physical memory pages until one attempts a write operation, triggering a page-level duplication.

- Process Table Entries (PID / PPID): The kernel tracks processes via an internal structural array, saving metadata like the current Process ID (PID) and Parent Process ID (PPID).

- PID vs fork() Return Value: fork() returns twice: it returns 0 inside the execution context of the child process, and returns the child's actual positive PID inside the parent process (or -1 on error).

- Parent / Child / Orphan / Zombie: A Parent creates a Child; an Orphan is a child whose parent died before it; a Zombie is a terminated process whose allocated resources are freed but its exit status remains uncollected by its parent.

- Adoption: In xv6, if a parent process terminates before its child, the init process (PID 1) automatically adopts the orphan child to ensure its eventual exit status is reaped.
### exit() and wait()

exit(status) halts execution and transitions a process to a zombie state; wait(&status) blocks the parent until a child exits, retrieving its status code and cleaning up its remaining process structures.

- Resources: Each process consumes limited kernel resources (such as process structures, page tables, and file descriptor slots) that must be reclaimed upon termination.

- Process Table Entry: The specific slot in the kernel's internal proc array holding execution state (RUNNABLE, SLEEPING, ZOMBIE), stack pointer, page table address, and open files.

- Arguments / Return values: System calls use standard CPU registers to pass configuration arguments in and pass execution success/error status flags back out to user space.
### exec()

Replaces the caller's entire user memory address space with a fresh executable image loaded directly from storage.

- File (ELF): Executable and Linkable Format; the standardized binary layout containing instructions, static data initialized sections, and metadata outlining memory arrangement.

- Return and Args: exec accepts an array of string argument pointers; it never returns on success, only returning -1 if the executable image fails to load.

- First Element: By standard convention, the first entry in the argument array (argv[0]) passed to exec represents the execution string or filename of the program itself.
### Example: Shell

```
+-------------------------------------------------------+
|                      SHELL LOOP                       |
+-------------------------------------------------------+
                           |
                           v
                     +-----------+
                     |  getcmd   | <----+  Read user input line 
                     +-----------+      
                           |            
                           v            
                     +-----------+      
                     |  fork()   |      
                     +-----------+      
                           |            
             +-------------+-------------+
             |                           |
             v  Child Process            v  Parent Process 
       +-----------+               +-----------+
       | parsecmd  |               |  wait()   |  Blocks until child ends 
       +-----------+               +-----------+
             |                           |
             v                           |
       +-----------+                     |
       |  runcmd   |                     |
       +-----------+                     |
             |                           |
             v                           |
       +-----------+                     |
       |  exec()   |  Overwrites child   |
       +-----------+                     |
             |                           |
             v                           |
       +-----------+                     |
       |  exit()   |  Releases memory    |
       +-----------+                     |
             |                           |
             +------------>| <-----------+  Parent wakes up, reaps zombie 
                           |
                           +-------------> Loop repeats
```
# 1.2 I/O and File Descriptors

- FD