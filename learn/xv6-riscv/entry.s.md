The entry point is at 0x80000000 set up in kernel.ld.

This \_entry function in [[entry.s]] sets up stack for the various harts. It uses the following math for each hart: 
$$ sp = stack0 + ((1024 * 4) * (mhartid + 1)) $$
Then it jumps to start() defined in [[start.c]]