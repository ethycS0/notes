# Sargantana DFI Coprocessor Architecture

## 1. Overview

The DFI coprocessor (DFIC) manages metadata and checks protected memory operations. A custom0 prefix supplies a static 16-bit identifier, and its associated memory instruction supplies the physical address, access size and ordinary memory outcome. DFIC combines this information to determine whether the protected operation may complete.

This document describes the proposed coprocessor side of [DFI Datapath Modifications](dfi_datapath.md). The [DFI policy](dfi_policy.md) remains the authority for security behavior. The prefix and memory instruction share one operation token, one architectural outcome and one fault/restart boundary at the prefix PC.

The existing L0 metadata caches and native HPDcache adapter are the reusable memory path. The token tracking, store reservations, ordering controls and processor completion interfaces described here are proposed additions. The older coprocessor and its separate four-bit token interfaces do not implement this design.

The initial scope recommends naturally aligned scalar integer loads and stores in normal RAM. The processor interfaces needed to establish load ordering, store success and safe recovery still require agreement. These dependencies are identified where they affect the design.

## 2. DFIC Organization

### Operation Tracking

Maintain an operation record for each owned nonzero token. Tokens are six bits wide: token `0` identifies ordinary operations, while values `1..63` identify protected operations. The prefix and suffix share the same token. The static ID16 identifies a load policy or store writer; it does not identify a particular execution of that instruction.

Each record retains the operation class, ID16, prefix GL index and associated suffix GL index. It also retains the PA and access width, ordinary outcome, metadata progress, final result and whether the operation has been killed or retired. For a load, metadata progress includes descriptor bounds, the scan cursor, required labels and accumulated match bits.

Keep room for one RDSet chunk in each load record. A chunk contains up to four IDs and its physical-lane mask. This permits early preparation without storing an entire variable-length RDSet.

The record remains owned while any processor event, metadata job or completion can still refer to it. A GL index alone is insufficient because GL locations are reused.

### Store-Update Storage

Maintain separate storage for reserved and committed RDTable updates. A STORE prefix reserves one entry when it is accepted. The entry eventually contains the writer ID, validated label addresses and write progress.

One entry covers the entire supported store, including both labels of an aligned eight-byte access. Capacity must therefore be reserved before the PA is known. A later doubleword store must not discover that a second unreserved entry is needed.

A committed entry can outlive the operation record and its token. After commitment, it retains its own ID, addresses and progress and no longer depends on the original GL entry.

### Metadata and Completion Paths

A shared metadata engine schedules requests to the existing RDTable, RDSmap and RDSet caches. Keep one metadata job active globally, including its held response. Retain the job's type, original owner, request payload and service cookie until completion.

Use a separate completion path to update the prefix's GL readiness and exception state. Pending results remain in operation records while the completion output is busy.

Address capture, ordinary completion capture and forwarding must continue while the metadata engine is stalled. A metadata miss must not prevent DFIC from receiving another admitted operation's PA or fault.

## 3. Prefix Admission and Pairing

### Request Acceptance

Create the operation record when the prefix actually allocates a GL entry. Use the allocation event described in the datapath proposal, rather than a valid bit held in the RR–EX register.

#### Relevant signals:

- `dfi_alloc.valid, dfi_alloc.token`: the token offered by DFIC before allocation.
- `dfi_req.valid, token, imm, gl_index`: the accepted prefix request.
- `dfi_req_credit`: proposed indication that DFIC can capture the candidate request and reserve its required storage.
- `alloc_fire_o`: the proposed GL allocation event.

Prefix allocation, token reservation and request capture succeed on the same edge. For a STORE prefix, update-entry reservation also occurs on that edge. If any required capacity is unavailable, stall allocation while allowing older memory operations, metadata updates and completions to progress.

Extract ID16 from `imm[27:12]` and class from `imm[29:28]`. Numeric class assignments still need agreement. The prepared instruction and its GL entry must capture the same token.

### Suffix Association

The datapath checks that the suffix follows at prefix PC + 4 and matches the prefix class. DFIC receives the accepted pair outcome and retains the suffix's GL ownership. PA reporting and ordinary completion then refer to this existing operation.

Keep the assigned value on `dfi_alloc.token` until the pair outcome is accepted. Once the prefix consumes the offer, that value must not be offered as free to another prefix. The proposed handling deasserts the offer's valid bit while retaining its token value; the suffix uses the saved token in `dfi_pending_q`.

Advancing the offer after association does not free the previous token. Its operation record remains owned until retirement or cancellation and all remaining references have drained.

#### Relevant signals:

- `dfi_pair_valid, dfi_pair_ready`: proposed pair-outcome handshake.
- `dfi_pair_token`: the pending operation's token.
- `dfi_pair_suffix_present, dfi_pair_suffix_gl`: associated suffix ownership, when present.
- `dfi_pair_outcome`: successful association, malformed pair or ordinary fault.
- `dfi_pair_exception`: original exception information for a faulted successor.

Normal association must be captured together with suffix allocation. The reserved operation record provides its destination, so pair acceptance must not depend on the metadata engine becoming idle.

### Malformed or Faulted Successors

A malformed pair must reach a fault outcome even if the successor cannot allocate or execute. The recommended extension allows the datapath to report that outcome from the live successor candidate before allocating it.

This matters for consecutive prefixes. If the first prefix waits for the second to allocate before accepting a malformed outcome, while the second waits for a new token, neither can progress. Reporting the terminal outcome first allows the original prefix to fault.

After a terminal pair outcome, retain a pending fault state until trap handling or cancellation. The offending successor remains blocked from execution; clearing association must not let it execute as an ordinary instruction.

Fetch bubbles do not imply a malformed pair. A real fetch fault at the expected suffix address carries its original exception. An older redirect cancels the pending pair so it cannot attach to an instruction at the new target. Fault-versus-malformation priority remains to be decided.

These terminal-outcome rules extend the current datapath proposal and need to be reflected there before implementation.

## 4. Address and Ordinary Memory Completion

### Physical-Address Reporting

Receive the registered `dfi_pa_q` channel described in the datapath proposal. Its fields are `valid`, `token`, `pa`, `mem_size` and `ex`. There is no ready signal: DFIC must capture one report every cycle independently of metadata activity.

A successful report provides the address and core access-size encoding. A fault report provides the original exception; ignore its PA and do not use it for an RDTable lookup.

Each admitted record has its own address destination. This allows several operations to report addresses while an earlier metadata request is stalled. The one-report-per-operation assumption applies only to the supported original scalar operations; fragmented accesses need additional handling.

### Access Width and Granules

Addresses use 64-bit containers, while the current standalone configuration uses 42-bit physical addresses. Validate physical high bits and configured ranges before narrowing an address.

Interpret `mem_size` together with the approved suffix opcode. For integer loads, bit 2 distinguishes unsigned forms rather than a larger access width. Derive the actual width of one, two, four or eight bytes before calculating metadata addresses.

#### Address calculations:

`last_exclusive = PA + byte_width`

`first_g = (PA - data_base) >> 2`

`last_g = (last_exclusive - 1 - data_base) >> 2`

`label_address(g) = rdt_base + 2*g`

Use widened arithmetic and validate the entire data footprint before subtraction or metadata access. Naturally aligned accesses of up to four bytes touch one granule; an aligned eight-byte access touches two. Every touched granule participates in the check or update.

A partial store assigns its ID to the whole touched granule under the current policy. Compiler treatment of objects sharing a granule remains a separate contract.

### Ordinary Completion

Provide a separate ordinary-memory completion event carrying the token, suffix GL identity and complete `exception_t`. A successful PA report does not establish that the memory access completed successfully.

For loads, derive the event from final memory processing, including cache misses, PMRQ replay and any required packing. A load to `x0` still needs an outcome even when it writes no register. Stores require a separate observation because they do not complete through the ordinary load-WB path.

#### Relevant signals:

- `dfi_ord_valid`: proposed final ordinary-outcome event.
- `dfi_ord_token, dfi_ord_suffix_gl`: the operation that completed.
- `dfi_ord_exception`: original ordinary fault, or no fault on definitive success.
- `dfi_issue_fire`: actual acceptance of the protected data request.

The proposed receiver accepts ordinary outcomes into reserved records without metadata backpressure. Retain the CPU request/tag association until its outcome and late responses are accounted for. The conservative issue scheme permits one protected data operation at a time, so one active protected memory-bridge record can suffice.

A translation fault may establish a final fault before normal WB. Any later copy must agree with that outcome and must not create another architectural completion.

The current CPU cache-response interface does not carry downstream error or aborted status. The precise meaning of definitive success therefore still depends on the supported platform fault model.

## 5. Load Verification

### Descriptor Lookup

A live LOAD prefix can begin RDSmap lookup as soon as its ID is accepted. PA availability is not required for this immutable metadata preparation.

Request the descriptor at `map_base + 8*ID`, requiring `ID < map_count`. Preserve the full returned descriptor. Its low 32 bits contain `begin`; its high 32 bits contain the exclusive `end`, measured in ID entries relative to `sets_base`.

Require `begin <= end <= sets_count`. Descriptor-content validation belongs to DFIC; the map cache returns the stored bits unchanged. An invalid descriptor is a service/configuration failure, not a DFI membership violation.

### RDSet Preparation and Scan

Set `cursor = begin`. For a nonempty set, DFIC may fetch one initial chunk before the data address or labels are ready. Retain that chunk in the operation record. Do not advance the comparison cursor or accumulate matches until the labels have been selected under the ordering rules below.

#### Chunk calculation:

`A = sets_base + 2*cursor`

`lane = (A >> 1) & 3`

`count = min(4-lane, end-cursor)`

`mask = ((1 << count)-1) << lane`

The request contains the descriptor bounds, cursor, count and physical address. It must stay within the list and one physical cache line. Lane selection uses the physical address because `sets_base` may be only two-byte aligned.

Once the required labels are available, use `dfi_rdset_match4` to compare them with the valid entries in each chunk. Accumulate one match bit per required label and advance the cursor by the number of compared entries.

The scan succeeds when every required label has matched. It may stop at that point. Reaching `end` with an unmatched label gives a membership failure. Duplicate IDs, unsorted sets and shared or overlapping storage remain legal; no sentinel or fixed maximum list length is introduced.

A valid empty RDSet requires no chunk request. After ordinary success and the required ordering conditions are established, membership in that empty set fails without unnecessary RDTable reads.

### Label Selection and Final Outcome

PA availability permits address calculation, but does not make an RDTable value suitable for the final check. Select labels only after the load's ordinary outcome and store-history conditions are established.

For a label obtained from L0, use the returned physical-lane mask to extract the requested ID. The selected label is not always in the low 16 bits. Bytes outside the mask and data from an errored response are unusable.

#### Successful load conditions:

- The pair is live and its footprint is supported and validated.
- The ordinary memory operation completed successfully.
- The selected labels describe the data observed by that load.
- Every required label matched the RDSet.
- No metadata or protocol failure prevents completion.

Only then may DFIC report successful prefix completion. A matching chunk or cache hit alone cannot authorize retirement.

An ordinary fault takes its ordinary-fault path without waiting for successful metadata verification. A final membership failure uses cause 18 at the prefix PC, subject to the agreed fault-priority rules.

## 6. Store Reservations and Committed Updates

### Reserving an Update

Reserve one update entry when an accepted STORE prefix allocates. Initially it contains the owner token and writer ID. After PA and size arrive, validate the complete footprint and retain the required label addresses.

A reserved entry is speculative. It cannot write RDTable or supply an authoritative label to a load. The associated store does not require RDSmap or RDSet verification.

Use the same storage entry when the store later commits. This avoids a second allocation that could fail after data effects have begun. Maintain a queue of committed-entry handles with matching capacity, so every reserved store can join commit order without needing another unreserved slot.

### Executing the Suffix Store

The suffix store must be able to execute while its prefix is still unready. Waiting for prefix readiness before granting the store permission would create a circular dependency: the prefix needs the store outcome before becoming ready.

Provide a separate protected-store permission based on the occupied GL head, live pair identity, validated footprint and reserved update entry. This observes the head prefix even when the normal ready-qualified GL outputs are empty.

Apply the permission to the exact suffix and record actual request acceptance once. The existing generic slot-1 store permission must not independently authorize or repeat that protected request when the prefix later becomes ready.

The permission must also participate in LSQ selection and dequeue. Suppressing only the final request valid can leave an ineligible load selected ahead of an older buffered store, preventing the store needed for progress from executing.

### Store Success and Retirement

Keep three events distinct: allowing store data to become irreversible, receiving its definitive ordinary outcome, and successfully retiring the protected pair.

Before any irreversible data effect, the entire update entry must be reserved and its footprint validated. Once data can no longer be undone, the processor must preserve the operation until successful commitment or defined terminal error handling. It cannot cancel the store as speculative work after its data escaped.

On definitive ordinary success, DFIC may complete the prefix. The dedicated store-pair retirement rule then requires the matching ready suffix, final ordinary success, live ownership and guaranteed update commitment. Retire both components together and promote the reserved entry to committed on the same edge.

#### Relevant signals:

- `dfi_mem_permit`: proposed permission for the exact protected suffix to issue.
- `dfi_issue_fire`: accepted data request.
- `dfi_ord_valid`: definitive ordinary outcome.
- `dfi_store_commit_ready`: the retained entry can become committed.
- `dfi_retire_valid`: successful pair retirement and update promotion.

The store rule must account for Sargantana's existing store-index, stall and fault paths. It cannot simply reuse the load-pair retirement predicate.

The exact irreversible point and definitive-success event remain unresolved processor/platform contracts. Existing request acceptance, store-packer completion and commit stalls do not alone prove that every possible ordinary fault is settled.

### Draining Committed Updates

A committed entry retains its ID, addresses and acknowledgement progress independently of the token and GL. Metadata latency therefore does not have to block store retirement. The token can later return to the free pool while its committed update remains pending.

Drain entries in commit order. For each entry, write every required label and wait for the complete L0 acknowledgement, including any resident-cache patch. Finish both labels of a doubleword before moving to the next entry.

Keep the whole entry available for forwarding until all its writes succeed. The baseline does not merge stores, skip overwritten entries or drain them out of order.

A failed write retains the entry and its progress and stops unsafe authorization. It does not undo the store's architectural commitment or turn the update into discarded speculative work. Native acknowledgement establishes local completion; later lower-memory failure handling still requires a platform contract.

## 7. Load Ordering and RDTable Forwarding

### Establishing the Load's Data History

The checked label must describe the data actually returned to the load. Waiting for load WB or in-order retirement alone does not prove this relationship.

The conservative proposal delays protected load data issue until its prefix is at the occupied GL head and older stores have settled. A proposed `dfi_hist_grant` confirms that all older CPU stores have completed the required commit and data-visibility steps. Their protected update commitments must already be recorded in DFIC.

The initial rule also waits for the HPDcache write buffer to become empty after older store sources have stopped supplying work. Buffer-empty alone is insufficient: an older store may still be held in the LSQ, store buffer, request pipeline or an outstanding transaction. Write-back stores also require proof that the CPU will observe their updated cache data.

Once the grant is accepted, block younger data-store publication through the protected load's retirement or cancellation. This includes ordinary stores that might otherwise issue early from a retirement slot. The platform must also prevent untracked writers or policy changes from invalidating the protected data and metadata during that interval.

After the load reports ordinary success, DFIC selects its final labels. No earlier tentative RDTable result may be reused unless its relationship to the observed data is proved.

This history grant and data-issue interlock are proposed additions to the datapath. They are not existing signals that DFIC can infer from a PA report.

### Pending-Update Forwarding

Search committed entries by the exact four-byte data granule, or its corresponding RDTable label address. Select the youngest matching entry in commit order. Token values, storage-slot numbers and wrapped GL indices do not establish age.

Only committed entries participate. A translated or reserved speculative store must not supply a label simply because its address matches.

Under the proposed history rule, every matching committed candidate is older than the load, and the load's data is after those stores. That relationship makes the youngest committed writer applicable. Without it, selecting the newest pending ID could check a new label against old data.

Copy a forwarded ID into the load record. The update entry may subsequently finish and be reclaimed without invalidating that copy. Partially drained two-label entries remain forwarding candidates for both of their granules until the whole entry completes.

If no committed entry matches, read RDTable through L0. Recheck forwarding before consuming the response, and do not overwrite a label already selected from a newer applicable update. An outstanding read still drains even when forwarding has satisfied the label.

Forwarding can run while another metadata job is active. At a simultaneous forwarding lookup and final update acknowledgement, preserve either the entry's ID or its successfully written memory representation; there must be no gap between them.

### Ordering Examples

If a store's data is visible while its label write is pending, the committed entry supplies the new ID to the load. Reading physical RDTable alone could return an old label.

If a speculative store has only reserved an entry, it supplies no label. A load cannot use that speculative ID while receiving older data, and an unresolved older store prevents the history grant.

For two committed stores to the same granule, forwarding chooses the later store. Ordered drain writes the earlier update first, preventing a delayed old update from overwriting a newer one.

To acquire the conservative buffer-empty checkpoint, temporarily stop offering new committed metadata writes, finish any already offered job and allow the shared write buffer to drain. The committed bank itself need not be empty. After the grant, metadata writes may resume because they materialize already committed labels without changing the load's data history.

## 8. Metadata-Service Interface

### Requests and Responses

Use the existing `dfi_l0_service` ports: RDTable on port 0, RDSmap on port 1 and RDSet on port 2. `dfi_l0_native` connects that service to the existing native adapter. Keep the planned DFI requester SID2 and the adapter's single outstanding native command.

The service request contains an opaque eight-bit cookie and physical metadata address. RDTable writes also carry `write`, `committed` and ID16. Map requests carry the selector; set requests carry the descriptor bounds, cursor and count.

The response echoes the cookie and write indication and returns error, data and mask. Retain a job record that identifies the original operation or committed update entry. The cookie, dynamic token, static ID, GL index and native transaction ID have different roles.

A committed-write response must address its retained update entry even if the original token has been reused. An operation response must not be applied to a later occupant of the same GL index.

The adapter handles native byte enables, write-lane positioning and read-data normalization. DFIC must not shift native data again. L0 response lane masks still determine which returned IDs are meaningful.

### Scheduling

Schedule fairly between committed writes, work needed by the current head load and immutable preparation for other admitted loads. RDT reads and writes share one service port, so DFIC must provide fairness between them.

Long RDSet scans yield after each chunk. An operation waiting for its PA, ordinary outcome or history grant must not hold the metadata engine while it waits.

Keep one job active globally, including an offered request or held response. Prepare a stable response destination before asserting request valid. Response capture must not wait for GL readiness or retirement.

### Cancellation and Errors

Hold a request stable while it is stalled. If its operation is killed after the request has been offered, retain ownership, drain the request and response, and discard its result. Cancellation does not permit a stalled valid payload to change.

An immutable read may still fill L0 after its original operation is killed within the same policy context. Its result cannot complete the killed operation. Context replacement waits for all old requests and fills to drain before invalidating the caches.

Check response ownership, kind and expected mask. An error or malformed live descriptor cannot authorize a protected operation. Transport and ownership faults require containment even if the original operation has been killed.

## 9. GL Completion and Retirement

### Independent Prefix Completion

DFIC reports a final prefix result through a separate ready/valid channel. It must not wait for the prefix to appear on GL's ready-qualified outputs, because those outputs remain empty while the prefix is unready.

The conservative proposal selects final completions when their prefix is the occupied GL head. Other results remain in their operation records.

#### Relevant signals:

- `dfi_head`: proposed observation of the occupied head prefix, independent of readiness.
- `dfi_cpl_valid, dfi_cpl_ready`: held completion handshake.
- `dfi_cpl_token, dfi_cpl_gl_idx`: prefix ownership.
- `dfi_cpl_exception`: final success or complete exception information.
- `dfi_cpl_discard`: consumption of a held completion for a canceled owner.

Before applying a completion, GL checks that the occupied entry is the live matching prefix and has not been killed. Update its exception flag, retained exception details and readiness together. Normal execution WB must not independently make a prefix ready.

Arbitrate DFIC completion with ordinary WB, allocation and branch exceptions. Preserve information that cannot be accepted, or prove that retaining the older fault is sufficient because younger work will be discarded. A fixed priority that loses a required exception is insufficient.

If a completion is killed while held, consume it without changing GL. Its payload remains stable through that discard handshake. A nonkilled ownership mismatch is a protocol error.

### Successful Retirement and Fault Disposal

For a successful load, retain the datapath proposal's joint retirement rule: prefix in slot 0, matching ready load in slot 1, same nonzero token and no exception or retirement stall. Both successful retirement masks become `2'b11`.

The stored final outcome and live ownership must also remain valid. Token equality alone does not establish that the ordinary access and DFI check succeeded.

Store retirement uses the separate success and update-commitment protocol above. Metadata writes may remain pending after the pair retires.

For a prefix fault, preserve the existing distinction between internal head removal and successful retirement: `retire_inst_o = 2'b01`, while `csr_retire = 2'b00`. Recovery removes the remaining pair and younger work.

Generate an explicit `dfi_trap_valid` event with the removed prefix's token and GL index. A later scan of occupied GL entries may no longer find that prefix, especially when a malformed operation has no suffix. This event marks the failed operation terminal while its outstanding work drains.

The prefix GL entry supplies the exception PC. Preserve the original ordinary cause and fault information, including `origin2`, `tinst` and `gva` where applicable.

## 10. Exceptions, Recovery and Token Release

### Exceptions and Interrupts

Malformed pairs use illegal-instruction cause 2. Proven DFI violations use cause 18. Ordinary memory faults retain their original cause and access information while using the prefix PC for restart.

The recommended priority is an ordinary fault before a same-pair membership failure. Priority involving malformed or faulted successors, and the DFI `tval` convention, still require agreement.

Current CSR behavior can retain cause 18, but its delegation masks exclude that cause and its `tval` selection produces zero. Whether to retain those software-visible choices or extend them remains a decision for the policy and core owners.

A prefix already carrying an allocation-time fault needs explicit handling. The recommendation is to leave it on the ordinary fault path with token 0 and no DFIC request. This changes the current unconditional prefix-request formula. Keeping normal admission instead requires an allocation-fault notification and a way to finish ownership without a suffix.

An interrupt must not split the pair. Before irreversible store effects, the processor may cancel the whole operation and restart at the prefix. Otherwise defer the interrupt until a safe whole-operation boundary. Preserve pending interrupts. Debug entry and single stepping require the same explicit boundary treatment.

### Flush and Recovery

Cancel protected operations as pairs. Partial recovery keeps older operations and kills only the discarded younger groups. Committed updates survive speculative flushes independently of their former tokens.

Before a store becomes irreversible, cancellation releases its speculative reservation without writing RDTable. After data escapes, the operation must finish its agreed commitment or enter terminal error handling; it cannot be discarded as though it had no effects.

Current branch recovery suppresses newly dispatching memory work but does not broadly flush the memory unit. The DFIC integration must remove killed queued operations, preserve surviving older work and suppress late victim events. Wiring branch kill directly into the existing broad memory flush would require additional preservation or replay.

For same-cycle events, a kill prevents a cancellable operation's new issue, successful completion or retirement. A surviving older store may still commit while younger work is killed. Native responses and committed update progress are retained through either case.

### Returning Tokens

A token returns to the free pool only after the operation has retired or received a trap/cancellation disposition, and no old processor, metadata or completion event can still refer to it.

Use a proposed `dfi_end_valid` event to confirm that the core's queued, registered, WB, replay and data-response references are finished. GL removal or WB alone does not provide that guarantee.

DFIC also waits for held pair outcomes, offered metadata jobs and held completions to drain. A committed update may remain because it has its own independent storage. Do not reuse the token on the same edge that closes its final old reference.

### Reset and Failure Handling

Warm reset and policy replacement require a coordinated drain: stop new admissions and data publication, cancel or finish speculative work, finish irreversible stores, drain committed updates and responses, synchronize the memory system, then invalidate L0.

Local L0 invalidation does not flush HPDcache, drain its write buffer or establish a memory fence. Replacing policy at the same physical addresses still requires synchronization and invalidation.

The native adapter reuses TID0 after reset. A destructive reset is safe only when downstream reset or drain prevents old responses from being mistaken for new transactions.

Metadata failures retain committed obligations and prevent unsafe authorization. The current CPU interface does not report arbitrary late downstream errors, so precise recovery from such failures cannot be assumed. Error transport or trusted terminal handling must be agreed before relying on asynchronous store-update completion.

## 11. Backpressure and Progress

Admission reserves the storage needed for every later unbackpressured event. Each token has its own PA and ordinary-outcome destination, and every accepted metadata job has a response destination.

The record logic must accept simultaneous events for different tokens and simultaneous updates to different fields of one token. A shared write-port priority chain must not silently drop PA, fault, retirement or kill information. If the selected storage cannot accept those events, add sufficient reserved capture capacity or change the producer handshake.

A full token pool stalls new protected prefixes. A full update bank stalls new STORE prefixes. Older memory processing, GL completion and committed drain continue so they can release those resources.

The update-bank capacity is a finite parameter independent of the token count. Entries that remain committed after token release still occupy capacity, including failed entries whose obligations have not been resolved.

Progress depends on responsive memory and fair arbitration. The history grant must not wait for the load's own WB or an empty committed bank. Store issue must not wait for successful prefix completion. Metadata response capture must not wait for GL retirement.

Under those conditions, finite RDSet scans complete, committed updates drain and canceled tokens eventually return to the pool. Arbitrarily long memory stalls do not justify dropping work, and no fixed cycle bound is claimed.

## 12. Supported Operations

The proposed baseline supports naturally aligned, 32-bit-encoded integer loads and stores of one, two, four or eight bytes in approved normal RAM.

Misaligned scalar accesses can be fragmented by VAGU into smaller requests. Supporting them requires the original footprint, fragment identities and complete fault/PA aggregation. An unaligned eight-byte access can touch three granules or cross into a noncontiguous physical page. The initial recommendation is explicit rejection before fragmentation or data effects, with the exception behavior agreed by the core and policy owners.

FP memory operations need the corresponding FPR completion and retirement paths. Vector operations additionally need masks, multiple addresses, partial completion and fault-first behavior. Neither becomes supported merely because it has a load/store memory type.

Compressed integer memory instructions may reuse scalar checking after their original PC and instruction length are preserved through decode and recovery. The suffix still starts at prefix PC + 4, but the following instruction is not necessarily at prefix PC + 8. Support requires explicit validation of this path.

MMIO must not use the normal-RAM protocol without accounting for irreversible device effects. Uncached normal RAM also requires its own visibility and error proof. Protected atomics and LR/SC remain unsupported until their read checks, conditional writes and labeling rules are defined.

Unsupported protected operations must be rejected before effects or prevented from running under an unapproved configuration. They must not silently fall back to token 0. Ordinary unprefixed operations remain allowed; their writes into protected data still require a policy decision.

The platform must protect metadata, initialize labels correctly and control untracked writers. Shared tokens alone do not enforce the policy's prohibition on direct control-flow entry into a suffix.

Metadata bounds checking also does not make arbitrary neighboring device memory safe: a small metadata request can cause a wider HPDcache line fill. Metadata regions and their containing lines must be approved for that access behavior.

## 13. Example Operation Flows

### Successful Protected Load

A prefix allocates with token 7, and its suffix receives the same token after pairing. DFIC starts map lookup while the CPU produces the PA. The descriptor supplies the set bounds, and an initial chunk may be retained before labels are known.

When the prefix reaches the GL head and the older-store conditions are satisfied, the processor grants the protected load permission to issue. Its eventual ordinary success arrives independently of GL readiness. DFIC then obtains labels from committed forwarding or RDTable and scans for both required matches.

After all required labels match, the prefix completion makes it ready. Prefix and suffix retire together. Token 7 remains owned until the core and all metadata/completion references have finished.

For example, with `data_base = 0x80000000`, `rdt_base = 0x90000002` and an LD at `PA = 0x80000008`, the required label addresses are `0x90000006` and `0x90000008`. Their physical lanes differ; extraction must follow the returned masks.

Likewise, `sets_base = 0xb0000002` and `cursor = 1` give address `0xb0000004`, lane 2. A list ending at index 7 starts with a two-entry chunk masked `1100`, followed by a four-entry chunk masked `1111`.

### Violating or Faulting Load

If an authoritative label is absent after the complete RDSet scan, DFIC reports cause 18 against the prefix. The pair and younger instructions do not retire successfully. Trap disposal and response drain precede token reuse.

If translation instead reports a page fault, DFIC preserves that ordinary exception and starts no RDTable lookup from the reported PA. The prefix can reach its fault path without a successful DFI check. An earlier offered immutable read still drains without changing the fault outcome.

### Store Retiring with Metadata Pending

A STORE prefix reserves an update entry. PA reporting fills its validated label addresses, and separate permission lets its suffix execute while the prefix is unready.

After definitive ordinary success, DFIC completes the prefix. Joint retirement promotes the same reserved entry to committed. The CPU may advance while label writes remain pending, and the original token may later be freed independently.

A younger load that has established the required data history can forward the writer ID from that committed entry. If the store updates two labels, keep the entry until both acknowledgements arrive. Failure on the second write preserves the first acknowledgement and the remaining obligation.

### Two Stores to the Same Granule

Stores with IDs `0xaaaa` and then `0xbbbb` commit to the same granule. A load whose data history is after both stores selects `0xbbbb`, regardless of token or storage-slot numbering.

Drain the earlier update before the later update. If the later entry is still pending, it overrides the earlier physical label. If both have completed, RDTable must provide the locally coherent later value.

### Recovery and Stalled Interfaces

A branch kill can cancel a younger reserved store while an older committed update continues draining. The younger reservation produces no label write. An offered metadata read for the killed operation is consumed and discarded before its token is reused.

If L0 stalls while three admitted operations report PAs on successive clocks, their separate records capture all three reports. New admission eventually stops when storage is exhausted; the reports already accepted are retained.

If a held prefix completion races a kill, consume it without updating GL. If a surviving prefix fault races an ordinary exception, arbitration must preserve the required fault information and update prefix details/readiness together.

## 14. Verification and Remaining Decisions

### Verification Requirements

Verify DFIC independently with explicit processor events and an independent data/provenance reference model. Then connect it to the real GL, LSQ, store buffer, PMRQ, CSR and CPU/DFI HPDcache requesters. An assumed history grant in a standalone test does not prove the processor can produce it correctly.

The essential properties are:

- No successful load retirement without final ordinary success and labels proven to describe its observed data.
- No RDTable write from a killed speculative store.
- Every successful committed store retains all required updates until completion.
- Committed updates survive speculative flushes and drain in order.
- No accepted PA, ordinary outcome or required exception is lost.
- No completion changes a reused GL entry, and no token is reused while old events can still arrive.
- No partial successful retirement of a protected pair.
- Resource exhaustion stalls admission while older progress paths remain active.
- Metadata and protocol errors cannot silently authorize an operation.

Directed tests should cover early metadata versus late WB, pending updates and stale reads, repeated stores to one granule, ordinary faults, empty and long sets, duplicate IDs, every physical starting lane, boundary refills, full storage, killed stalled requests, GL-index wrap, completion/flush races and malformed pairs without suffix entries.

Use the actual L0 RAM wrappers and native adapter. Existing L0 evidence covers the isolated service; the proposed processor ordering and completion protocols still require integration tests. No implementation or new hardware verification is implied by this document.

### Decisions Still Required

The processor team must define how the history grant proves older-store visibility and how protected issue permission changes LSQ selection. The exact irreversible store point, definitive success event and handling of late memory failures must be established before allowing store retirement with metadata pending.

The core and DFIC interfaces also need agreement on request credits, terminal pair outcomes, already-faulted prefix admission, ordinary completion, GL arbitration, trap disposal and final processor-reference release.

Policy decisions remain for fault priority, DFI `tval` and delegation, unsupported-operation rejection, initial labels, partial-object layouts, unprotected writers and atomic semantics. Interrupt and debug behavior must preserve the whole-operation boundary.

The chosen rules should later be reflected in `dfi_datapath.md`. The existing policy requirements for ID16, four-byte granules, pre-commit load checks and committed-only store labels remain intact.
