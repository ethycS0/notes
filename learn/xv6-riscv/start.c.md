This is where the basic hart is setup to go from machine mode to supervisor mode. `stack0` is defined here where `NCPU * 4096` is allocated. This allows `NCPU` stacks to be present and we can now write in C.

## start()
This function sets up every hart privellege switch from machine to supervisor. The process is to first set `mpp` to supervisor so `mret` will change prvillege to supervisor. `mepc` is set to `main()` so that 