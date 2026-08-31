[[INDEX]] | [[addressing-and-caches/virtual-and-physical-addressing]] | [[addressing-and-caches/cpu-caches]]

Cache side-channel attacks exploit timing differences in memory access latency (cache hits vs. cache misses) to extract sensitive information across security boundaries. In transient execution attacks like Meltdown, the cache serves as a microarchitectural covert channel: transient instructions transmit secret data into cache residency, and the attacker receives the transmitted data by measuring memory access times.

![Flush+Reload Attack Sequence](flush-reload-attack.jpg)

## Flush+Reload Attack Technique

The primary cache side-channel technique used in Meltdown is **Flush+Reload**, introduced by Yuval Yarom and Katrina Falkner in 2014. Flush+Reload operates on shared memory pages (such as shared libraries or a user-allocated probe array) across three distinct execution phases:

1. **Flush Phase:** The attacker uses the unprivileged x86 instruction `clflush` to evict a target memory line from all levels of the CPU cache hierarchy (L1, L2, LLC), ensuring it is strictly uncached.
2. **Victim / Transient Execution Phase:** The attacker executes the transient sequence. The CPU transiently reads a secret kernel byte $S$ and multiplies it by a stride offset (e.g., $S \times 4096$). The CPU accesses `probe_array[S * 4096]`, bringing that specific array page line into the L1 data cache.
3. **Reload & Time Phase:** The attacker iterates through all 256 candidate array offsets ($0$ to $255$), measuring the access time for each line using high-resolution hardware timers (`rdtsc` / `rdtscp`). The line corresponding to offset $S \times 4096$ returns in ~4 cycles (Cache Hit), while all other 255 lines take ~200+ cycles (Cache Miss). The attacker immediately infers that the secret byte $S$ equals the index of the fast cache line.

The 4096-byte stride (one 4KB page per candidate byte value) is critical to prevent spatial prefetching noise. Modern hardware prefetchers automatically fetch adjacent cache lines within a 64-byte block; allocating each candidate value on a separate 4096-byte physical page ensures that fetching `probe_array[S]` only brings page $S$ into the cache, keeping all other 255 candidate pages uncached.

## Alternative Cache Attack Mechanisms

While Meltdown uses Flush+Reload due to its high reliability and signal-to-noise ratio, other cache side-channel techniques exist when shared memory or `clflush` instructions are restricted:

- **Evict+Reload:** Replaces the hardware `clflush` instruction with targeted memory accesses that pollute the target cache set, forcing the CPU's replacement policy to evict the target line before measuring reload times.
- **Prime+Probe:** Operates without shared memory by filling (priming) an entire CPU cache set with attacker data, waiting for victim execution, and then measuring access times (probing) to identify which cache sets were evicted by victim activity.

---
*Return to [[INDEX]]*
